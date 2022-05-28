# DAX in ext2 filesystem


### msync syscall

```
/mm/msync.c

/ * MS_SYNC syncs the entire file - including mappings. */
SYSCALL_DEFINE3(msync, unsigned long, start, size_t, len, int, flags)
    vfs_fsync_range(file, fstart, fend, 1);
        file->f_op->fsync(file, start, end, datasync);
        == ext2_fsync();
            generic_file_fsync(file, start, end, datasync);
                __generic_file_fsync(struct file *file, loff_t start, loff_t end, int datasync)
                    file_write_and_wait_range(file, start, end);
                        __filemap_fdatawrite_range(mapping, lstart, lend, WB_SYNC_ALL);
                            do_writepages(mapping, &wbc);
                                mapping->a_ops->writepages(mapping, wbc);

ext2_dax_writepages()
    dax_writeback_mapping_range(mapping, mapping->host->i_sb->s_bdev, wbc);
        dax_writeback_one(&xas, dax_dev, mapping, entry);
            dax_flush(dax_dev, page_address(pfn_to_page(pfn)), size);
                arch_wb_cache_pmem(addr, size);
                    clean_cache_range(addr, size);
                        for (p = (void *)((unsigned long)addr & ~clflush_mask); p < vend; p += x86_clflush_size)
                            clwb(p);

static const struct address_space_operations ext2_dax_aops = {
	.writepages		= ext2_dax_writepages,
	.direct_IO		= noop_direct_IO,
	.set_page_dirty		= noop_set_page_dirty,
	.invalidatepage		= noop_invalidatepage,
};

void ext2_set_file_ops(struct inode *inode)
{
	inode->i_op = &ext2_file_inode_operations;
	inode->i_fop = &ext2_file_operations;
	if (IS_DAX(inode))
		inode->i_mapping->a_ops = &ext2_dax_aops;
	else if (test_opt(inode->i_sb, NOBH))
		inode->i_mapping->a_ops = &ext2_nobh_aops;
	else
		inode->i_mapping->a_ops = &ext2_aops;
}
```

### nvdimm driver
```
/drivers/nvdimm/pmem.h

/drivers/nvdimm/pmem.c

pmem_dax_direct_access()
    __pmem_direct_access()

pmem_copy_from_iter()
    copy_from_iter_flushcache(addr, bytes, i);
        _copy_from_iter_flushcache(addr, bytes, i);
            iterate_and_advance(i, bytes, v,
                __copy_from_user_flushcache((to += v.iov_len) - v.iov_len,
                    v.iov_base, v.iov_len),
                memcpy_page_flushcache((to += v.bv_len) - v.bv_len, v.bv_page,
                    v.bv_offset, v.bv_len),
                memcpy_flushcache((to += v.iov_len) - v.iov_len, v.iov_base,
                    v.iov_len)
            )


pmem_rw_page(struct block_device *bdev, sector_t sector, struct page *page, unsigned int op);
    pmem_do_bvec(pmem, page, hpage_nr_pages(page) * PAGE_SIZE, 0, op, sector);
        if (!op_is_write(op)) {
            read_pmem(page, off, pmem_addr, len);
            flush_dcache_page(page);
        } else {
            flush_dcache_page(page);
            write_pmem(pmem_addr, page, off, len);
        }  

write_pmem()
    memcpy_flushcache()
        __memcpy_flushcache(dst, src, cnt);
            /* assembly code ... */
            while (size >= 32) {
                asm("movq    (%0), %%r8\n"
                "movq   8(%0), %%r9\n"
                "movq  16(%0), %%r10\n"
                "movq  24(%0), %%r11\n"
                "movnti  %%r8,   (%1)\n"
                "movnti  %%r9,  8(%1)\n"
				...
                :: "r" (source), "r" (dest)
                : "memory", "r8", "r9", "r10", "r11");
                dest += 32;
                source += 32;
                size -= 32;
            }


static void write_pmem(void *pmem_addr, struct page *page,
		unsigned int off, unsigned int len)
{
	unsigned int chunk;
	void *mem;

	while (len) {
		mem = kmap_atomic(page);
		chunk = min_t(unsigned int, len, PAGE_SIZE);
		memcpy_flushcache(pmem_addr, mem + off, chunk);
		kunmap_atomic(mem);
		len -= chunk;
		off = 0;
		page++;
		pmem_addr += PAGE_SIZE;
	}
}

static blk_status_t read_pmem(struct page *page, unsigned int off,
		void *pmem_addr, unsigned int len)
{
	unsigned int chunk;
	unsigned long rem;
	void *mem;

	while (len) {
		mem = kmap_atomic(page);
		chunk = min_t(unsigned int, len, PAGE_SIZE);
		rem = memcpy_mcsafe(mem + off, pmem_addr, chunk);
		kunmap_atomic(mem);
		if (rem)
			return BLK_STS_IOERR;
		len -= chunk;
		off = 0;
		page++;
		pmem_addr += PAGE_SIZE;
	}
	return BLK_STS_OK;
}

static const struct block_device_operations pmem_fops = {
	.owner =		THIS_MODULE,
	.rw_page =		pmem_rw_page,
	.revalidate_disk =	nvdimm_revalidate_disk,
};

static const struct dax_operations pmem_dax_ops = {
	.direct_access = pmem_dax_direct_access,
	.copy_from_iter = pmem_copy_from_iter,
	.copy_to_iter = pmem_copy_to_iter,
};

```

### ext2 
```
# /fs/ext2/file.c

ext2_file_read_iter()
    ext2_dax_read_iter()
        dax_iomap_rw();
            iomap_apply();
                dax_iomap_actor();
                    dax_direct_access();
                    dax_copy_from_iter();
                    dax_copy_to_iter();
                        dax_dev->ops->copy_from_iter(dax_dev, pgoff, addr, bytes, i);


static const struct vm_operations_struct ext2_dax_vm_ops = {
	.fault		= ext2_dax_fault,
	/*
	 * .huge_fault is not supported for DAX because allocation in ext2
	 * cannot be reliably aligned to huge page sizes and so pmd faults
	 * will always fail and fail back to regular faults.
	 */
	.page_mkwrite	= ext2_dax_fault,
	.pfn_mkwrite	= ext2_dax_fault,
};

static int ext2_file_mmap(struct file *file, struct vm_area_struct *vma)
{
	if (!IS_DAX(file_inode(file)))
		return generic_file_mmap(file, vma);

	file_accessed(file);
	vma->vm_ops = &ext2_dax_vm_ops;
	return 0;
}
```
