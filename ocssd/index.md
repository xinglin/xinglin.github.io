# Open-Channel SSD


- In Linux pblk implementation, a line corresponds to a chunk which is one erase block. A LUN is a PU. A group is a channel. 

- Eventually, pblk calls pblk_submit_read() and pblk_submit_write() for reads/writes to the device. 

- Read code path (from the media) in pblk  

```
/* pblk-read.c */

pblk_submit_read()
  pblk_read_ppalist_rq();
  or pblk_read_rq(); 
    pblk_lookup_l2p_seq();           // get physical addresses
  pblk_submit_io(pblk, rqd);
    nvm_submit_io(dev, rqd);
      dev->ops->submit_io(dev, rqd);
```

- Write code path in pblk

```
pblk_write_to_cache() {

 	for (i = 0; i < nr_entries; i++) {
		void *data = bio_data(bio);

		w_ctx.lba = lba + i;

		pos = pblk_rb_wrap_pos(&pblk->rwb, bpos + i);
		pblk_rb_write_entry_user(&pblk->rwb, data, w_ctx, pos);

		bio_advance(bio, PBLK_EXPOSED_PAGE_SIZE);
	}

	pblk_write_should_kick(struct pblk *pblk);
}

/* pblk-rb.c */
void pblk_rb_write_entry_user(struct pblk_rb *rb, void *data,
	      struct pblk_w_ctx w_ctx, unsigned int ring_pos)
{
	__pblk_rb_write_entry(rb, data, w_ctx, entry);
	pblk_update_map_cache(pblk, w_ctx.lba, entry->cacheline);
}

/* pblk-core.c */
void pblk_write_should_kick(struct pblk *pblk)
{
	unsigned int secs_avail = pblk_rb_read_count(&pblk->rwb);

	if (secs_avail >= pblk->min_write_pgs_data)
		pblk_write_kick(pblk);
}

pblk_write_kick()
 wake_up_process(pblk->writer_ts);
  pblk_write_ts() 	/* pblk-write.c */
   pblk_submit_write(pblk, $secs_left);

pblk_submit_write(struct pblk* pblk, int *secs_left)
 pblk_rb_read_to_bio(&pblk->rwb, rqd, pos, secs_to_sync, secs_avail)
 pblk_submit_io_set(pblk, rqd);
  pblk_setup_w_rq(pblk, rqd, &erase_ppa);   // will call pblk_map_rq() to allocate new physical pages; will also call pblk_line_erase() to first erase if needed.
  pblk_submit_io(pblk, rqd);
    nvm_submit_io(dev, rqd);
      dev->ops->submit_io(dev, rqd);
```

- Garbage Collection code path in pblk
Erase operations are done during writes (when flushing the write buffer to the media).  

```
/* pblk-gc.c */
int pblk_gc_init(struct pblk *pblk)
{
 gc->gc_ts = kthread_create(pblk_gc_ts, pblk, "pblk-gc-ts");
 gc->gc_writer_ts = kthread_create(pblk_gc_writer_ts, pblk, 
							"pblk-gc-writer-ts");
 gc->gc_reader_ts = kthread_create(pblk_gc_reader_ts, pblk,
							"pblk-gc-reader-ts");

/* Workqueue that reads valid sectors from a line and submit them to the
 * GC writer to be recycled.
 */
gc->gc_line_reader_wq = alloc_workqueue("pblk-gc-line-reader-wq",
			WQ_MEM_RECLAIM | WQ_UNBOUND, PBLK_GC_MAX_READERS);

/* Workqueue that prepare lines for GC */
gc->gc_reader_wq = alloc_workqueue("pblk-gc-line_wq",
				WQ_MEM_RECLAIM | WQ_UNBOUND, 1);
}


pblk_gc_ts()
 pblk_gc_run()
  pblk_gc_get_victim_line()
  pblk_gc_reader_kick()

pblk_gc_reader_ts()
 pblk_gc_read()
  pblk_gc_line()
   pblk_gc_line_prepare_ws()
    pblk_gc_line_ws();
     pblk_submit_read_gc(pblk, gc_rq);
      read_ppalist_rq_gc();           // find all nonempty pages.
      pblk_submit_io_sync();
     list_add_tail(&gc_rq->list, &gc->w_list);	// add nonempty pages to the gc->w_list
     pblk_gc_writer_kick(&pblk->gc);		// kick writer to do writes
    kref_put(&line->ref, pblk_line_put);        // add this line back to free list, if there is no reference.
pblk_gc_writer_ts()
 pblk_gc_write(pblk)
  pblk_write_gc_to_cache(pblk, gc_rq);
   pblk_rb_write_entry_gc();
    __pblk_rb_write_entry();
    pblk_update_map_gc();
```

- Scratch space

```
/* pblk-init.c */

pblk_make_rq()
  if (bio_op(bio) == REQ_OP_DISCARD) 
	pblk_discard(pblk, bio);

  if (bio_data_dir(bio) == READ) {
	blk_queue_split(q, &bio);
	pblk_submit_read(pblk, bio);
  } else {
	if (pblk_get_secs(bio) > pblk_rl_max_io(&pblk->rl))
		blk_queue_split(q, &bio);

	pblk_write_to_cache(pblk, bio, PBLK_IOTYPE_USER);
  }

/* get phyiscal address fro logical address */
ppa = pblk_trans_map_get(pblk, lba)

int pblk_lookup_l2p_seq(struct pblk *pblk, struct ppa_addr *ppas,
			 sector_t blba, int nr_secs, bool *from_cache)
void pblk_lookup_l2p_rand(struct pblk *pblk, struct ppa_addr *ppas,
			  u64 *lba_list, int nr_secs)

/* pblk-read.c */

pblk_submit_read()
  pblk_read_ppalist_rq();
  or pblk_read_rq(); 
    pblk_lookup_l2p_seq();           // get physical addresses
  pblk_submit_io(pblk, rqd);
    nvm_submit_io(dev, rqd);
      dev->ops->submit_io(dev, rqd);

/* pblk-write.c */
pblk_submit_write(struct pblk* pblk, int *secs_left)
 pblk_rb_read_to_bio(&pblk->rwb, rqd, pos, secs_to_sync, secs_avail)
 pblk_submit_io_set(pblk, rqd);
  pblk_setup_w_rq(pblk, rqd, &erase_ppa);   // will call pblk_map_rq() to allocate physical pages
  pblk_submit_io(pblk, rqd);

pblk_setup_w_rq()
 pblk_map_rq()       // allocate physical pages for new writes.

/* Allocate new physical pages from an erase block */
u64 pblk_alloc_page(struct pblk *pblk, struct pblk_line *line, int nr_secs)


/* lightnvm.c */
static struct nvm_dev_ops nvme_nvm_dev_ops = {
	.identity		= nvme_nvm_identity,

	.get_bb_tbl		= nvme_nvm_get_bb_tbl,
	.set_bb_tbl		= nvme_nvm_set_bb_tbl,

	.get_chk_meta		= nvme_nvm_get_chk_meta,

	.submit_io		= nvme_nvm_submit_io,
	.submit_io_sync		= nvme_nvm_submit_io_sync,

	.create_dma_pool	= nvme_nvm_create_dma_pool,
	.destroy_dma_pool	= nvme_nvm_destroy_dma_pool,
	.dev_dma_alloc		= nvme_nvm_dev_dma_alloc,
	.dev_dma_free		= nvme_nvm_dev_dma_free,
};

/* pblk-cache.c */
pblk_write_to_cache() {

 	for (i = 0; i < nr_entries; i++) {
		void *data = bio_data(bio);

		w_ctx.lba = lba + i;

		pos = pblk_rb_wrap_pos(&pblk->rwb, bpos + i);
		pblk_rb_write_entry_user(&pblk->rwb, data, w_ctx, pos);

		bio_advance(bio, PBLK_EXPOSED_PAGE_SIZE);
	}

	pblk_write_should_kick(struct pblk *pblk);
}

/* pblk-rb.c */
void pblk_rb_write_entry_user(struct pblk_rb *rb, void *data,
	      struct pblk_w_ctx w_ctx, unsigned int ring_pos)
{
	__pblk_rb_write_entry(rb, data, w_ctx, entry);
	pblk_update_map_cache(pblk, w_ctx.lba, entry->cacheline);
}

/* pblk-core.c */
void pblk_write_should_kick(struct pblk *pblk)
{
	unsigned int secs_avail = pblk_rb_read_count(&pblk->rwb);

	if (secs_avail >= pblk->min_write_pgs_data)
		pblk_write_kick(pblk);
}

pblk_write_kick()
 wake_up_process(pblk->writer_ts);
  pblk_write_ts() 	/* pblk-write.c */
   pblk_submit_write(pblk, $secs_left);



/* pblk-map.c */
pblk_map_rq()
 pblk_map_page_data()
  pblk_alloc_page()

/* pblk-core.c */
pblk_blk_erase_sync
pblk_blk_erase_async


pblk_setup_w_rq()
 pblk_map_rq();
 or pblk_map_erase_rq();
   pblk_map_page_data()
     if (pblk_line_is_full(line)) 
       line = pblk_line_replace_data(pblk);
         pblk_line_erase(pblk, new);

pblk_map_rq()
 pblk_map_page_data()

pblk_map_erase_rq()
 pblk_map_page_data()
  if (pblk_line_is_full(line)) 
    line = pblk_line_replace_data(pblk);
```

```
/* pblk.h */

/* a line is an erase block. */

enum {
	/* Line Types */
	PBLK_LINETYPE_FREE = 0,
	PBLK_LINETYPE_LOG = 1,
	PBLK_LINETYPE_DATA = 2,

	/* Line state */
	PBLK_LINESTATE_NEW = 9,
	PBLK_LINESTATE_FREE = 10,
	PBLK_LINESTATE_OPEN = 11,
	PBLK_LINESTATE_CLOSED = 12,
	PBLK_LINESTATE_GC = 13,
	PBLK_LINESTATE_BAD = 14,
	PBLK_LINESTATE_CORRUPT = 15,

};
```

