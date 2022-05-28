# Read syscall implementation in Linux Kernel

 

### Interactions among vfs, page cache, and ext2 on serving a read request
![](../sysread.png)

# stack trace for ext2_readpages() in ext2
```
[   84.037457] Call Trace:
[   84.037458]  dump_stack+0x46/0x5b
[   84.037460]  ext2_readpages+0x3e/0x90
[   84.037464]  read_pages+0x71/0x1a0
[   84.037470]  ? __do_page_cache_readahead+0x1c9/0x1e0
[   84.037472]  __do_page_cache_readahead+0x1c9/0x1e0
[   84.037474]  ondemand_readahead+0x171/0x2b0
[   84.037478]  ? pagecache_get_page+0x30/0x2c0
[   84.037481]  ? __kernel_text_address+0xe/0x30
[   84.037483]  generic_file_read_iter+0x875/0xda0
[   84.037412]  ext2_file_read_iter+0x4c/0xe0
[   84.037488]  new_sync_read+0x12e/0x1d0
[   84.037490]  vfs_read+0x91/0x130
[   84.037492]  ksys_read+0x52/0xc0
[   84.037494]  do_syscall_64+0x4f/0x100
[   84.037496]  entry_SYSCALL_64_after_hwframe+0x44/0xa9

[   84.037379] Call Trace:
[   84.037405]  dump_stack+0x46/0x5b
[   84.037412]  ext2_file_read_iter+0x4c/0xe0
[   84.037419]  new_sync_read+0x12e/0x1d0
[   84.037426]  vfs_read+0x91/0x130
[   84.037428]  ksys_read+0x52/0xc0
[   84.037431]  do_syscall_64+0x4f/0x100
[   84.037438]  entry_SYSCALL_64_after_hwframe+0x44/0xa9

```

# read/write syscall definitions
read/write syscalls are defined in fs/read_write.c, as following. 
```
SYSCALL_DEFINE3(read, unsigned int, fd, char __user *, buf, size_t, count)
{
	return ksys_read(fd, buf, count);
}
SYSCALL_DEFINE3(write, unsigned int, fd, const char __user *, buf,
		size_t, count)
{
	return ksys_write(fd, buf, count);
}
```
# Read syscall call stack

```
ksys_read(fd, buf, count);
  vfs_read(f.file, buf, count, &pos);
    __vfs_read(file, buf, count, pos);
    	if (file->f_op->read)
		return file->f_op->read(file, buf, count, pos);
	else if (file->f_op->read_iter)
		return new_sync_read(file, buf, count, pos);
	else
		return -EINVAL;

/* Take ext2 as an example. ext2 does not define read() but defines read_iter(). */
const struct file_operations ext2_file_operations = {
	.llseek		= generic_file_llseek,
	.read_iter	= ext2_file_read_iter,
	.write_iter	= ext2_file_write_iter,
	.unlocked_ioctl = ext2_ioctl,
#ifdef CONFIG_COMPAT
	.compat_ioctl	= ext2_compat_ioctl,
#endif
	.mmap		= ext2_file_mmap,
	.open		= dquot_file_open,
	.release	= ext2_release_file,
	.fsync		= ext2_fsync,
	.get_unmapped_area = thp_get_unmapped_area,
	.splice_read	= generic_file_splice_read,
	.splice_write	= iter_file_splice_write,
};

new_sync_read()
  call_read_iter(filp, &kiocb, &iter);
    file->f_op->read_iter(kio, iter);
```

# ext2_file_read_iter()
```
ext2_file_read_iter(): ext2_file_operations.read_iter
  generic_file_read_iter()
    generic_file_buffered_read()
      find_get_page()
      page_cache_sync_readahead()
      or page_cache_async_readahead()
        ondemand_readahead()
          __do_page_cache_readahead()
            read_pages()
              mapping->a_ops->readpages(filp, mapping, pages, nr_pages);
              or mapping->a_ops->readpage(filp, page);

/* ext2 defines both readpage() and readpages(). */
const struct address_space_operations ext2_aops = {
	.readpage		= ext2_readpage,
	.readpages		= ext2_readpages,
	.writepage		= ext2_writepage,
	.write_begin		= ext2_write_begin,
	.write_end		= ext2_write_end,
	.bmap			= ext2_bmap,
	.direct_IO		= ext2_direct_IO,
	.writepages		= ext2_writepages,
	.migratepage		= buffer_migrate_page,
	.is_partially_uptodate	= block_is_partially_uptodate,
	.error_remove_page	= generic_error_remove_page,
};

/* The IO request is submitted to the block layer by calling mpage_bio_submit(). */
ext2_readpages()
    mpage_readpages()
        mpage_bio_submit()
```
  

```
/*
 * __do_page_cache_readahead() actually reads a chunk of disk.  It allocates
 * the pages first, then submits them for I/O. This avoids the very bad
 * behaviour which would occur if page allocations are causing VM writeback.
 * We really don't want to intermingle reads and writes like that.
 *
 * Returns the number of pages requested, or the maximum amount of I/O allowed.
 */
unsigned int __do_page_cache_readahead(struct address_space *mapping,
		struct file *filp, pgoff_t offset, unsigned long nr_to_read,
		unsigned long lookahead_size)
{
    ...

	/*
	 * Preallocate as many pages as we will need.
	 */
	for (page_idx = 0; page_idx < nr_to_read; page_idx++) {
		pgoff_t page_offset = offset + page_idx;

		if (page_offset > end_index)
			break;

		page = xa_load(&mapping->i_pages, page_offset);
		if (page && !xa_is_value(page)) {
			/*
			 * Page already present?  Kick off the current batch of
			 * contiguous pages before continuing with the next
			 * batch.
			 */
			if (nr_pages)
				read_pages(mapping, filp, &page_pool, nr_pages,
						gfp_mask);
			nr_pages = 0;
			continue;
		}

		page = __page_cache_alloc(gfp_mask);
		if (!page)
			break;
		page->index = page_offset;
		list_add(&page->lru, &page_pool);
		if (page_idx == nr_to_read - lookahead_size)
			SetPageReadahead(page);
		nr_pages++;
	}  


	/*
	 * Now start the IO.  We ignore I/O errors - if the page is not
	 * uptodate then the caller will launch readpage again, and
	 * will then handle the error.
	 */
	if (nr_pages)
		read_pages(mapping, filp, &page_pool, nr_pages, gfp_mask);
}

static int read_pages(struct address_space *mapping, struct file *filp,
		struct list_head *pages, unsigned int nr_pages, gfp_t gfp)
{
	struct blk_plug plug;
	unsigned page_idx;
	int ret;

	blk_start_plug(&plug);

	if (mapping->a_ops->readpages) {
		ret = mapping->a_ops->readpages(filp, mapping, pages, nr_pages);
		/* Clean up the remaining pages */
		put_pages_list(pages);
		goto out;
	}

	for (page_idx = 0; page_idx < nr_pages; page_idx++) {
		struct page *page = lru_to_page(pages);
		list_del(&page->lru);
		if (!add_to_page_cache_lru(page, mapping, page->index, gfp))
			mapping->a_ops->readpage(filp, page);
		put_page(page);
	}
	ret = 0;

out:
	blk_finish_plug(&plug);

	return ret;
}
```

# IO Path from Understanding the Linux Kernel
The following explanation is from the book, Understanding the Linux Kernel. It seems that the implementions of Linux VFS and specific filesystems have evolved that the explanation from the book is not accurate. 

In Linux, when a file is opened by a process, its file object will be added to the process control block (struct task_struct). The process control block has a pointer, struct files_struct* files, to keep track of all opened files. Within the files_struct, there exists struct fdtable, storing file objects for all file descriptors. A file descriptor is an index into this file object array of type struct file. Each file object (struct file) has a pointer to a file operations object (struct file_operations), to invoke file system specific operations for that file. 

When a read() is called from the process for a file descriptor, it will invoke the sys_read(), which calls the read() operation for that file object (file->f_op->read(...)). 

```

struct task_struct {
	...
	
	/* Open file information: */
	struct files_struct		*files;
};

/*
 * Open file table structure
 */
struct files_struct {
  /*
   * read mostly part
   */
	atomic_t count;
	bool resize_in_progress;
	wait_queue_head_t resize_wait;

	struct fdtable __rcu *fdt;
	struct fdtable fdtab;
  /*
   * written part on a separate cache line in SMP
   */
	spinlock_t file_lock ____cacheline_aligned_in_smp;
	unsigned int next_fd;
	unsigned long close_on_exec_init[1];
	unsigned long open_fds_init[1];
	unsigned long full_fds_bits_init[1];
	struct file __rcu * fd_array[NR_OPEN_DEFAULT];
};

struct fdtable {
	unsigned int max_fds;
	struct file __rcu **fd;      /* current fd array */
	unsigned long *close_on_exec;
	unsigned long *open_fds;
	unsigned long *full_fds_bits;
	struct rcu_head rcu;
};

struct file {
	union {
		struct llist_node	fu_llist;
		struct rcu_head 	fu_rcuhead;
	} f_u;
	struct path		f_path;
	struct inode		*f_inode;	/* cached value */
	const struct file_operations	*f_op;

	/*
	 * Protects f_ep_links, f_flags.
	 * Must not be taken from IRQ context.
	 */
	spinlock_t		f_lock;
	enum rw_hint		f_write_hint;
	atomic_long_t		f_count;
	unsigned int 		f_flags;
	fmode_t			f_mode;
	struct mutex		f_pos_lock;
	loff_t			f_pos;
	struct fown_struct	f_owner;
	const struct cred	*f_cred;
	struct file_ra_state	f_ra;

	...
} __randomize_layout
  __attribute__((aligned(4)));	/* lest something weird decides that 2 is OK */


```


[vfs]: http://www.tldp.org/LDP/khg/HyperNews/get/fs/vfstour.html

