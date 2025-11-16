# Ethernet Adventures

## Issue
It was discovered that the SIS190 Ethernet driver works to detect xb360 Ethernet hardware. However since trying it on a Trinity Slim console
I have been having issues with and want to try to solve:
- "link on unknown mode"
- Fatal driver crashes that require ifupdown to be run again to recover

I see in logs:
```
Nov 15 19:22:29 gentoo360 kernel: sis190 0000:01:07.0 eth0: 0000:01:07.0: Xenon PCI Fast Ethernet adapter at (____ptrval____) (IRQ: 76), 7c:1e:52:0d:f2:af
Nov 15 19:22:29 gentoo360 kernel: sis190 0000:01:07.0 eth0: GMII mode.
Nov 15 19:22:29 gentoo360 kernel: sis190 0000:01:07.0 eth0: Enabling Auto-negotiation
Nov 15 19:22:59 gentoo360 kernel: sis190 0000:01:07.0 eth0: mii ext = 9198
Nov 15 19:22:59 gentoo360 kernel: sis190 0000:01:07.0 eth0: mii lpa=c5e1 adv=01e1 exp=000f
Nov 15 19:22:59 gentoo360 kernel: sis190 0000:01:07.0 eth0: link on unknown mode
```

This will happen many times during any given boot. If the system is not under high network load there will be no crashing
and the system is responsive. I had sessions last several hours if all I was doing is running an SSH client at a prompt.
However stressing the link while in "unknown" state causes these crashes:
```
Nov 15 17:08:06 gentoo360 kernel: CPU: 0 UID: 0 PID: 2278 Comm: git Not tainted 6.17.7-xenon #2 VOLUNTARY
Nov 15 17:08:06 gentoo360 kernel: Hardware name: Xenon Game Console Xenon 0x710800 Xenon
Nov 15 17:08:06 gentoo360 kernel: Call Trace:
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffef950] [c000000000b18190] dump_stack_lvl+0x8c/0xc8 (unreliable)
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffef980] [c000000000291100] warn_alloc+0x140/0x1d8
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffefa20] [c000000000291e44] __alloc_pages_slowpath.constprop.0+0xcac/0xde0
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffefba0] [c000000000292240] __alloc_frozen_pages_noprof+0x2c8/0x330
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffefc20] [c0000000002922c0] __alloc_pages_noprof+0x18/0x38
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffefc40] [c000000000293a1c] __page_frag_alloc_align+0xd4/0x1c8
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffefc80] [c0000000008dfc54] __netdev_alloc_skb+0xb4/0x270
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffefcd0] [c000000000802264] sis190_rx_fill.isra.0+0xbc/0x278
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffefdc0] [c0000000008028a8] sis190_irq+0x488/0x728
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffefeb0] [c0000000000e55fc] __handle_irq_event_percpu+0x74/0x190
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffeff40] [c0000000000e573c] handle_irq_event_percpu+0x24/0x90
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffeff70] [c0000000000eda08] handle_percpu_irq+0x78/0xb0
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffeffa0] [c0000000000e470c] handle_irq_desc+0x4c/0x78
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffeffc0] [c0000000000100f0] __do_irq+0x38/0x78
Nov 15 17:08:06 gentoo360 kernel: [c00000000ffeffe0] [c0000000000107a8] __do_IRQ+0x68/0xe0
Nov 15 17:08:06 gentoo360 kernel: [c0000000093ae460] [c008000002cc92a0] 0xc008000002cc92a0
Nov 15 17:08:06 gentoo360 kernel: [c0000000093ae4a0] [c0000000000108a0] do_IRQ+0x80/0xe8
Nov 15 17:08:06 gentoo360 kernel: [c0000000093ae4d0] [c000000000008040] hardware_interrupt_common_virt+0x220/0x230
Nov 15 17:08:06 gentoo360 kernel: ---- interrupt: 500 at ZSTD_compressBlock_doubleFast+0x8ec/0x4570 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: NIP:  c0080000032ac13c LR: c008000002c503c8 CTR: 0000000000000001
Nov 15 17:08:06 gentoo360 kernel: REGS: c0000000093ae500 TRAP: 0500   Not tainted  (6.17.7-xenon)
Nov 15 17:08:06 gentoo360 kernel: MSR:  9000000000009032 <SF,HV,EE,ME,IR,DR,RI>  CR: 44888888  XER: 20000000
Nov 15 17:08:06 gentoo360 kernel: IRQMASK: 0 \x0aGPR00: 0000000000000030 c0000000093ae7a0 c008000003328000 00000000032d4262 \x0aGPR04: 0000000003295476 c000000019fa0149 000000000000f384 c000000019fa00c9 \x0aGPR08: c008000002c56380 c000000016c5fffe c24b893b5f76688b c008000002c96380 \x0aGPR12: c000000019fa0000 c000000001250000 c000000019fa00c8 c000000019fb0000 \x0aGPR16: 0000>
Nov 15 17:08:06 gentoo360 kernel: NIP [c0080000032ac13c] ZSTD_compressBlock_doubleFast+0x8ec/0x4570 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: LR [c008000002c503c8] 0xc008000002c503c8
Nov 15 17:08:06 gentoo360 kernel: ---- interrupt: 500
Nov 15 17:08:06 gentoo360 kernel: [c0000000093ae7a0] [c0000000093ae8a0] 0xc0000000093ae8a0 (unreliable)
Nov 15 17:08:06 gentoo360 kernel: [c0000000093ae890] [c00800000329982c] ZSTD_buildSeqStore+0x3ec/0x530 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093ae960] [c00800000329f540] ZSTD_compressBlock_internal+0x58/0x208 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aea10] [c0080000032a07c0] ZSTD_compressContinue_internal+0x518/0xdc0 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aebd0] [c0080000032a2980] ZSTD_compressEnd_public+0x38/0x210 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aec30] [c0080000032a5e6c] ZSTD_compressStream2+0xadc/0xb60 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aecd0] [c0080000032a60b0] ZSTD_compress2+0x78/0xe8 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aed40] [c008000003290b14] zstd_compress_cctx+0xd4/0x108 [zstd_compress]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aeda0] [c008000002705ad8] zstd_compress+0x50/0xb8 [zram]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aedd0] [c0080000027002fc] zcomp_compress+0x7c/0xd0 [zram]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aee40] [c008000002704404] zram_write_page+0x1dc/0x4f8 [zram]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aeef0] [c008000002704de4] zram_submit_bio+0x6c4/0x820 [zram]
Nov 15 17:08:06 gentoo360 kernel: [c0000000093aefb0] [c000000000596adc] __submit_bio+0x16c/0x310
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af040] [c000000000596db0] submit_bio_noacct_nocheck+0x130/0x328
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af0b0] [c00000000058f564] submit_bio_wait+0x8c/0x118
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af130] [c0000000002a1b54] swap_writepage_bdev_sync+0x15c/0x1a0
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af1f0] [c0000000002a2bd8] swap_writeout+0x1f8/0x3f0
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af230] [c00000000022cd4c] shrink_folio_list.isra.0+0xe04/0xed8
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af470] [c00000000022e3b4] shrink_lruvec+0x524/0xd08
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af620] [c00000000022eee8] shrink_node+0x350/0xa80
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af6f0] [c000000000230460] do_try_to_free_pages+0x120/0x698
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af790] [c0000000002311f0] try_to_free_pages+0x138/0x198
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af850] [c000000000291614] __alloc_pages_slowpath.constprop.0+0x47c/0xde0
Nov 15 17:08:06 gentoo360 kernel: [c0000000093af9d0] [c000000000292240] __alloc_frozen_pages_noprof+0x2c8/0x330
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afa50] [c000000000292994] __folio_alloc_noprof+0x1c/0x40
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afa70] [c000000000218254] __filemap_get_folio+0x15c/0x648
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afb50] [c0000000003ee174] ext4_da_write_begin+0x11c/0x3b8
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afbf0] [c0000000002110e4] generic_perform_write+0x194/0x3a0
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afcb0] [c0000000003d2a4c] ext4_buffered_write_iter+0x84/0x168
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afcf0] [c0000000002dac10] vfs_write+0x2f8/0x568
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afdb0] [c0000000002db054] ksys_write+0x84/0x140
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afe10] [c00000000002286c] system_call_exception+0x14c/0x298
Nov 15 17:08:06 gentoo360 kernel: [c0000000093afe50] [c00000000000b354] system_call_common+0xf4/0x258
Nov 15 17:08:06 gentoo360 kernel: ---- interrupt: c00 at 0x7fffad05f3b4
Nov 15 17:08:06 gentoo360 kernel: NIP:  00007fffad05f3b4 LR: 00007fffad05f410 CTR: 0000000000000000
Nov 15 17:08:06 gentoo360 kernel: REGS: c0000000093afe80 TRAP: 0c00   Not tainted  (6.17.7-xenon)
Nov 15 17:08:06 gentoo360 kernel: MSR:  900000000200d032 <SF,HV,VEC,EE,PR,ME,IR,DR,RI>  CR: 2404444d  XER: 00000000
Nov 15 17:08:06 gentoo360 kernel: IRQMASK: 0 \x0aGPR00: 0000000000000004 00007fffd6aac950 00007fffad1f7100 0000000000000003 \x0aGPR04: 000000012c658610 0000000000001000 0000000000000000 0000000000000000 \x0aGPR08: 0000000000000000 0000000000000000 0000000000000000 0000000000000000 \x0aGPR12: 0000000000000000 00007fffad3db520 0000000000000000 00007fffd6aad838 \x0aGPR16: 0000>
Nov 15 17:08:06 gentoo360 kernel: NIP [00007fffad05f3b4] 0x7fffad05f3b4
Nov 15 17:08:06 gentoo360 kernel: LR [00007fffad05f410] 0x7fffad05f410
Nov 15 17:08:06 gentoo360 kernel: ---- interrupt: c00
Nov 15 17:08:06 gentoo360 kernel: Mem-Info:
Nov 15 17:08:06 gentoo360 kernel: active_anon:638 inactive_anon:1385 isolated_anon:32\x0a active_file:304 inactive_file:174 isolated_file:0\x0a unevictable:115 dirty:13 writeback:0\x0a slab_reclaimable:309 slab_unreclaimable:1167\x0a mapped:328 shmem:115 pagetables:133\x0a sec_pagetables:0 bounce:0\x0a kernel_misc_reclaimable:0\x0a free:14 free_pcp:85 free_cma:0
Nov 15 17:08:06 gentoo360 kernel: Node 0 active_anon:40832kB inactive_anon:88640kB active_file:19456kB inactive_file:11136kB unevictable:7360kB isolated(anon):2048kB isolated(file):0kB mapped:20992kB dirty:832kB writeback:0kB shmem:7360kB kernel_stack:7904kB pagetables:8512kB sec_pagetables:0kB all_unreclaimable? no Balloon:0kB
Nov 15 17:08:06 gentoo360 kernel: Normal free:896kB boost:0kB min:2688kB low:3328kB high:3968kB reserved_highatomic:0KB free_highatomic:0KB active_anon:40896kB inactive_anon:88832kB active_file:19648kB inactive_file:11904kB unevictable:7360kB writepending:1280kB present:491520kB managed:469632kB mlocked:0kB bounce:0kB free_pcp:5440kB local_pcp:512kB free_cma:0kB
Nov 15 17:08:06 gentoo360 kernel: lowmem_reserve[]: 0 0
Nov 15 17:08:06 gentoo360 kernel: Normal: 1*64kB (U) 1*128kB (U) 3*256kB (U) 0*512kB 0*1024kB 0*2048kB 0*4096kB 0*8192kB 0*16384kB = 960kB
Nov 15 17:08:06 gentoo360 kernel: Node 0 hugepages_total=0 hugepages_free=0 hugepages_surp=0 hugepages_size=16384kB
Nov 15 17:08:06 gentoo360 kernel: 950 total pagecache pages
Nov 15 17:08:06 gentoo360 kernel: 341 pages in swap cache
Nov 15 17:08:06 gentoo360 kernel: Free swap  = 21107392kB
Nov 15 17:08:06 gentoo360 kernel: Total swap = 21495616kB
Nov 15 17:08:06 gentoo360 kernel: 7680 pages RAM
Nov 15 17:08:06 gentoo360 kernel: 0 pages HighMem/MovableOnly
```

## Testing

Since my user experience taught me that stressing the link out with large downloads or complex tasks could reproduce the problem
I took it as a starting point. When I actually looked at my upload and download figures on the console itself it was only adding up to
10Mbps maximum, getting 1MB or less total actual throughput. I didn't record these poor figures but was able to easily spot it using iperf3.

So I then looked to a userland solution. I found some old posts about other SIS190 users on some motherboards having success with:
- Setting the NIC MTU to 1492 (`ip link set eth0 mtu 1492`)
This didn't help in my case.
https://forums.opensuse.org/t/sis191-not-working/34240
https://community.ipfire.org/t/driver-issue-on-sis191-and-high-interrupts/10610

So I tried using `ethtool` to set the following parameters as an informed decision:
`ethtool -s eth0 autoneg off duplex full speed=100`

I first tried the above after early boot time which would fatally crash the driver. On reload, it still had the exact same issue and log messages as above:
```
Nov 15 19:22:29 gentoo360 kernel: sis190 0000:01:07.0 eth0: Enabling Auto-negotiation
Nov 15 19:22:59 gentoo360 kernel: sis190 0000:01:07.0 eth0: mii ext = 9198
Nov 15 19:22:59 gentoo360 kernel: sis190 0000:01:07.0 eth0: mii lpa=c5e1 adv=01e1 exp=000f
Nov 15 19:22:59 gentoo360 kernel: sis190 0000:01:07.0 eth0: link on unknown mode
```

Deciding that is expected behavior of hot-setting these parameters on most NICs I pivoted to trying to use Gentoo's `netifrc` to set the same options during
early boot and NIC **bringup**. Even more interestingly the same log messages occur. The plot thickens.

At this point I had two options: explore reviving the old `xenon_net` driver, or modify the SIS190 driver. Neither seemed like a great option but after a few hours
trying to port up the old driver I gave up on it. I then thought well, if I know and can rely on every 360 console's network *specification* being the same, then why
not force the SIS190 driver to set 100Mbps full duplex mode on initialization. I then also thought this would be a decent trade off because the POST looks for Ethernet NIC
problems on boot as per https://xenonlibrary.com/wiki/Errors#Secondary_Error_Codes

## Change to SIS190 driver (TLDR)

A single line change to the SIS190 driver clears up the fatal crashes. During high load (iperf3 tests every 10 minutes for 8 hours)
sometimes the driver still infrequently crashes. However now it seems to be transient instead of blocking like before.

```
{ 0, 0x04000800 | 0x00001000, "FALLBACK: 100 Mbps Full Duplex" } // Added as catchall for PowerPC Xenon
```

A non-blocking crash from after making this change:
```
Nov 16 00:02:00 gentoo360 kernel: CPU: 0 UID: 0 PID: 0 Comm: swapper/0 Not tainted 6.17.8-xenon #2 VOLUNTARY
Nov 16 00:02:00 gentoo360 kernel: Hardware name: Xenon Game Console Xenon 0x710800 Xenon
Nov 16 00:02:00 gentoo360 kernel: Call Trace:
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffef910] [c000000000a9b724] dump_stack_lvl+0x84/0xe8 (unreliable)
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffef940] [c00000000027d388] warn_alloc+0x114/0x1bc
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffef9e0] [c00000000027d630] __alloc_pages_slowpath.constprop.0+0x200/0xd8c
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffefb80] [c00000000027e490] __alloc_frozen_pages_noprof+0x2d4/0x300
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffefc00] [c00000000027e4d4] __alloc_pages_noprof+0x18/0x38
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffefc20] [c00000000027fc64] __page_frag_alloc_align+0xb0/0x1bc
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffefc60] [c000000000878f3c] __netdev_alloc_skb+0xb4/0x218
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffefcb0] [c0000000007a8390] sis190_rx_fill.isra.0+0xc0/0x2a4
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffefd90] [c0000000007a8a54] sis190_irq+0x4e0/0x748
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffefea0] [c0000000000de8e0] __handle_irq_event_percpu+0x88/0x1ac
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffeff40] [c0000000000dea28] handle_irq_event_percpu+0x24/0x88
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffeff70] [c0000000000e64e4] handle_percpu_irq+0x78/0xb0
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffeffa0] [c0000000000dda6c] handle_irq_desc+0x5c/0x7c
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffeffc0] [c00000000000ff9c] __do_irq+0x38/0x74
Nov 16 00:02:00 gentoo360 kernel: [c00000000ffeffe0] [c0000000000107d4] __do_IRQ+0x6c/0xc0
Nov 16 00:02:00 gentoo360 kernel: [c00000000114fb30] [c000000001090e00] 0xc000000001090e00
Nov 16 00:02:00 gentoo360 kernel: [c00000000114fb70] [c0000000000108a8] do_IRQ+0x80/0xe4
Nov 16 00:02:00 gentoo360 kernel: [c00000000114fba0] [c000000000008040] hardware_interrupt_common_virt+0x220/0x230
Nov 16 00:02:00 gentoo360 kernel: ---- interrupt: 500 at do_idle+0x9c/0x184
Nov 16 00:02:00 gentoo360 kernel: NIP:  c0000000000b4680 LR: c0000000000b4678 CTR: c00000000086b984
Nov 16 00:02:00 gentoo360 kernel: REGS: c00000000114fbd0 TRAP: 0500   Not tainted  (6.17.8-xenon)
Nov 16 00:02:00 gentoo360 kernel: MSR:  900000000200b032 <SF,HV,VEC,EE,FP,ME,IR,DR,RI>  CR: 28000480  XER: 00000000
Nov 16 00:02:00 gentoo360 kernel: IRQMASK: 0 \x0aGPR00: c0000000000b46c0 c00000000114fe70 c000000000d88000 0000000000000000 \x0aGPR04: 0000000000000108 4000000000000000 000000001cc80000 c000000001250000 \x0aG>
Nov 16 00:02:00 gentoo360 kernel: NIP [c0000000000b4680] do_idle+0x9c/0x184
Nov 16 00:02:00 gentoo360 kernel: LR [c0000000000b4678] do_idle+0x94/0x184
Nov 16 00:02:00 gentoo360 kernel: ---- interrupt: 500
Nov 16 00:02:00 gentoo360 kernel: [c00000000114fe70] [c0000000000b46c0] do_idle+0xdc/0x184 (unreliable)
Nov 16 00:02:00 gentoo360 kernel: [c00000000114fed0] [c0000000000b4998] cpu_startup_entry+0x48/0x4c
Nov 16 00:02:00 gentoo360 kernel: [c00000000114ff00] [c00000000000d52c] rest_init+0xe4/0xe8
Nov 16 00:02:00 gentoo360 kernel: [c00000000114ff30] [c0000000010011d4] start_kernel+0x5d0/0x8d8
Nov 16 00:02:00 gentoo360 kernel: [c00000000114ffe0] [c00000000000c608] start_here_common+0x1c/0x20
Nov 16 00:02:00 gentoo360 kernel: Mem-Info:
Nov 16 00:02:00 gentoo360 kernel: active_anon:62 inactive_anon:590 isolated_anon:0\x0a active_file:1233 inactive_file:1793 isolated_file:0\x0a unevictable:115 dirty:130 writeback:190\x0a slab_reclaimable:529 >
Nov 16 00:02:00 gentoo360 kernel: Node 0 active_anon:3968kB inactive_anon:37760kB active_file:78912kB inactive_file:114752kB unevictable:7360kB isolated(anon):0kB isolated(file):0kB mapped:16448kB dirty:8320k>
Nov 16 00:02:00 gentoo360 kernel: Normal free:960kB boost:0kB min:2688kB low:3328kB high:3968kB reserved_highatomic:0KB free_highatomic:0KB active_anon:3968kB inactive_anon:37312kB active_file:78912kB inactiv>
Nov 16 00:02:00 gentoo360 kernel: lowmem_reserve[]: 0 0
Nov 16 00:02:00 gentoo360 kernel: Normal: 1*64kB (U) 1*128kB (U) 2*256kB (U) 0*512kB 0*1024kB 0*2048kB 0*4096kB 0*8192kB 0*16384kB = 704kB
Nov 16 00:02:00 gentoo360 kernel: Node 0 hugepages_total=0 hugepages_free=0 hugepages_surp=0 hugepages_size=16384kB
Nov 16 00:02:00 gentoo360 kernel: 3225 total pagecache pages
Nov 16 00:02:00 gentoo360 kernel: 22 pages in swap cache
Nov 16 00:02:00 gentoo360 kernel: Free swap  = 21412992kB
Nov 16 00:02:00 gentoo360 kernel: Total swap = 21495616kB
Nov 16 00:02:00 gentoo360 kernel: 7680 pages RAM
Nov 16 00:02:00 gentoo360 kernel: 0 pages HighMem/MovableOnly
Nov 16 00:02:00 gentoo360 kernel: 342 pages reserved
```

I am not sure what this is telling me really. Does the NIC need a periodic interrupt? There are no syslogs to go along with the crash unlike my initial adventure. As everything is now working without
fatally crashing I am happy, but I may revisit it.
