[  155.865893] Unable to handle kernel NULL pointer dereference at virtual address 000000000000001c
[  155.865935] Mem abort info:
[  155.865945]   ESR = 0x0000000096000045
[  155.865956]   EC = 0x25: DABT (current EL), IL = 32 bits
[  155.865970]   SET = 0, FnV = 0
[  155.865981]   EA = 0, S1PTW = 0
[  155.865991]   FSC = 0x05: level 1 translation fault
[  155.866002] Data abort info:
[  155.866010]   ISV = 0, ISS = 0x00000045, ISS2 = 0x00000000
[  155.866022]   CM = 0, WnR = 1, TnD = 0, TagAccess = 0
[  155.866033]   GCS = 0, Overlay = 0, DirtyBit = 0, Xs = 0
[  155.866046] user pgtable: 4k pages, 39-bit VAs, pgdp=0000000080cb1000
[  155.866060] [000000000000001c] pgd=0000000000000000, p4d=0000000000000000, pud=0000000000000000
[  155.866094] Internal error: Oops: 0000000096000045 [#3] PREEMPT SMP
[  155.866110] Modules linked in: rfcomm cmac algif_hash aes_arm64 aes_generic algif_skcipher af_alg bnep binfmt_misc brcmfmac_wcc snd_soc_wm8960 regmap_i2c vc4 brcmfmac hci_uart btbcm brcmutil snd_soc_hdmi_codec bluetooth bcm2835_codec(C) drm_display_helper bcm2835_isp(C) bcm2835_v4l2(C) rpi_hevc_dec v3d cec bcm2835_mmal_vchiq(C) cfg80211 snd_soc_simple_card vc_sm_cma(C) drm_dma_helper videobuf2_vmalloc v4l2_mem2mem snd_soc_bcm2835_i2s videobuf2_dma_contig gpu_sched videobuf2_memops drm_shmem_helper videobuf2_v4l2 snd_soc_simple_card_utils snd_soc_core ecdh_generic ecc snd_compress drm_kms_helper videodev rfkill snd_bcm2835(C) snd_pcm_dmaengine videobuf2_common libaes raspberrypi_hwmon snd_pcm mc snd_timer i2c_brcmstb i2c_bcm2835 snd raspberrypi_gpiomem nvmem_rmem uio_pdrv_genirq uio sch_fq_codel i2c_dev drm zram lz4_compress fuse drm_panel_orientation_quirks backlight nfnetlink ip_tables x_tables ipv6
[  155.866525] CPU: 1 UID: 1000 PID: 1640 Comm: speaker-test Tainted: G      D  C         6.12.58-v8+ #1
[  155.866548] Tainted: [D]=DIE, [C]=CRAP
[  155.866559] Hardware name: Raspberry Pi 4 Model B Rev 1.5 (DT)
[  155.866569] pstate: 60000005 (nZCv daif -PAN -UAO -TCO -DIT -SSBS BTYPE=--)
[  155.866586] pc : snd_pcm_playback_open+0x4c/0xb0 [snd_pcm]
[  155.866654] lr : snd_pcm_playback_open+0x40/0xb0 [snd_pcm]
[  155.866706] sp : ffffffc085743a70
[  155.866716] x29: ffffffc085743a70 x28: ffffffc085743c80 x27: 0000000000000000
[  155.866744] x26: 0000000000000006 x25: 0000000000000002 x24: 0000000000000002
[  155.866771] x23: 0000000007400002 x22: ffffffe13ec76c78 x21: ffffff8043b5d600
[  155.866797] x20: ffffff80427a8780 x19: 0000000000000000 x18: 0000000000000000
[  155.866822] x17: 0000000000000000 x16: ffffffe1a55a88a0 x15: 0000000000000000
[  155.866847] x14: ffffffffffffffff x13: ffffff8041af5029 x12: ffffffc085743cc4
[  155.866873] x11: 000000081358e89e x10: 0000000000000002 x9 : ffffffe13ec7a070
[  155.866898] x8 : ffffff8044048f00 x7 : 70304430436d6370 x6 : 000000000000002c
[  155.866923] x5 : ffffffe13ec88d30 x4 : 0000000000000000 x3 : 0000000000000000
[  155.866948] x2 : 0000000000000001 x1 : 0000000000000000 x0 : ffffff8043b5d600
[  155.866974] Call trace:
[  155.866985]  snd_pcm_playback_open+0x4c/0xb0 [snd_pcm]
[  155.867036]  snd_open+0xac/0x1c0 [snd]
[  155.867095]  chrdev_open+0xbc/0x218
[  155.867121]  do_dentry_open+0x140/0x4e0
[  155.867141]  vfs_open+0x34/0xf8
[  155.867161]  path_openat+0x2b8/0xe78
[  155.867178]  do_filp_open+0x88/0x140
[  155.867193]  do_sys_openat2+0xbc/0xf0
[  155.867212]  __arm64_sys_openat+0x68/0xb8
[  155.867231]  invoke_syscall+0x50/0x120
[  155.867253]  el0_svc_common.constprop.0+0x48/0xf8
[  155.867273]  do_el0_svc+0x28/0x40
[  155.867291]  el0_svc+0x30/0x100
[  155.867308]  el0t_64_sync_handler+0x13c/0x158
[  155.867324]  el0t_64_sync+0x190/0x198
[  155.867345] Code: 97fdd269 d2800001 52800022 aa0003f5 (b9001c22) 
[  155.867361] ---[ end trace 0000000000000000 ]---