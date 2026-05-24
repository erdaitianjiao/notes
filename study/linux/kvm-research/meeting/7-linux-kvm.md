# pKVM/KVM Patch

## pkvm引入、host降权、初始VMX/EPT/MMU框架

c800bda7f378a1d6e32e0fdb76aa65394c5d35de	pKVM on Intel introduction	pKVM on Intel introduction
71be221646fe798c049fa0ebbc33b2b68fa5756c	pkvm: x86: Introduce CONFIG_PKVM_INTEL	pkvm: x86: Introduce CONFIG_PKVM_INTEL
013b184e0e9ae0ad7a937e0a0040fe6440ccacd3	KVM: VMX: Refactor for setup_vmcs_config	KVM: VMX: Refactor for setup_vmcs_config
7273332481b7c0bd0222ea1716f0cc467b939ff9	pkvm: x86: Add vmx capability check and vmx config setup	pkvm: x86: Add vmx capability check and vmx config setup
f96016e00ec7aef1fd8c39c0fbb48039d54e549a	pkvm: x86: Add pCPU env setup	pkvm: x86: Add pCPU env setup
47c0c048aceafba8ecc1b8878201690c4aa40e13	pkvm: x86: Add basic setup for host vcpu	pkvm: x86: Add basic setup for host vcpu
d5b286f5a4e4b22933c530e6153476030198d422	pkvm: x86: Introduce pkvm_host_deprivilege_cpus	pkvm: x86: Introduce pkvm_host_deprivilege_cpus
1e23286631a14693c52982db408c1425a3c4b63a	pkvm: x86: Allocate vmcs and msr bitmap pages for host vcpu	pkvm: x86: Allocate vmcs and msr bitmap pages for host vcpu
145cbbbbbdc78a057105222c2a4716b38f978f25	pkvm: x86: Initailize vmcs guest state area for host vcpu	pkvm: x86: Initailize vmcs guest state area for host vcpu
e9b595ef9a87d95b24572ae3c086d96e07c8b5e2	pkvm: x86: Initialize vmcs host state area for host vcpu	pkvm: x86: Initialize vmcs host state area for host vcpu
dcc3afacb729e6b83dfbb300a99badbc5509c534	pkvm: x86: Initialize vmcs control fields for host vcpu	pkvm: x86: Initialize vmcs control fields for host vcpu
953013299aaa3804ebadea50ff0b384a4365a3bd	pkvm: x86: Define empty debug functions for hypervisor	pkvm: x86: Define empty debug functions for hypervisor
304cecd867f77afe4a6ab91bb433f6a804f5189c	pkvm: x86: Add vmexit handler for host vcpu	pkvm: x86: Add vmexit handler for host vcpu
d3e06235bfd5560efb09541f84f0e9c2cf8d758d	pkvm: x86: Add private vmx_ops.h for pKVM	pkvm: x86: Add private vmx_ops.h for pKVM
34a5920dad5a73382376220ee8e58f44679d904d	pkvm: x86: Add pKVM retpoline.S	pkvm: x86: Add pKVM retpoline.S
3282557218edc005471ed2283e9e928ddd5e820c	pkvm: x86: Build pKVM runtime as an independent binary	pkvm: x86: Build pKVM runtime as an independent binary
facc2107e6f866e791f597b18ec5ce1a3f591cde	pkvm: x86: Deprivilege host OS	pkvm: x86: Deprivilege host OS
af719e2e75085921b5f997cb2e44eaf05be8d5f8	pkvm: x86: Stub CONFIG_DEBUG_LIST in pKVM	pkvm: x86: Stub CONFIG_DEBUG_LIST in pKVM
11a21ef399ddb053138e9a10d19cdc34c20d78ac	Isolate pKVM & host	Isolate pKVM & host
92fc19fbdac61ff68ce995dadedf21d3e460f7a8	pkvm: x86: Define hypervisor runtime VA/PA APIs	pkvm: x86: Define hypervisor runtime VA/PA APIs
dcfa967bfc7d00e78b1fbae79c4e746824880d8b	pkvm: x86: Add memset lib	pkvm: x86: Add memset lib
829c12ead2666f9c3365e029a99b0728a27a5c2f	pkvm: x86: Add buddy page allocator	pkvm: x86: Add buddy page allocator
d418e453aaa66808691a2d0191520d82eb2d475d	pkvm: x86: Generate pkvm_constants.h for pKVM initialization	pkvm: x86: Generate pkvm_constants.h for pKVM initialization
150233f46d8843e49b37445fe80925c57d51ec45	pkvm: x86: Calculate total reserve page numbers	pkvm: x86: Calculate total reserve page numbers
78c84ea39dc30972de29b627557382ac8f16043a	pkvm: x86: Reserve memory for pKVM	pkvm: x86: Reserve memory for pKVM
66ec2b5c8d3e7cef604600e70d8abba30719907c	pkvm: x86: Early alloc from reserved memory	pkvm: x86: Early alloc from reserved memory
92d4abe85decc8b5a412b67ba5533a70da7a8519	pkvm: x86: Introduce general page table management framework	pkvm: x86: Introduce general page table management framework
1f31dbd0e32d3b2a0724123f1c1e42944c006627	pkvm: x86: Initialize MMU/EPT configuration	pkvm: x86: Initialize MMU/EPT configuration
73aa500dfd13ba6568c58f215e8a2aab62229433	pkvm: x86: Add early allocator based mm_ops	pkvm: x86: Add early allocator based mm_ops
1d0328d7f7ee6c44c06809d49a3cf043892c3049	pkvm: x86: Introduce MMU pgtable support	pkvm: x86: Introduce MMU pgtable support
ba73a0e96c2949a45b1c41a30c84c4a4c04de06e	pkvm: x86: Add global pkvm_hyp pointer	pkvm: x86: Add global pkvm_hyp pointer
82d011fca1ba88e4c8f8b630663500d272d50c8f	pkvm: x86: Add init-finalise hypercall	pkvm: x86: Add init-finalise hypercall
10a05a5c15a6395572628efe4dc18567c933f3c1	pkvm: x86: Create MMU pgtable in init-finalise hypercall	pkvm: x86: Create MMU pgtable in init-finalise hypercall
2b87485756f2abad541a1b659fb3e86149463f53	pkvm: x86: Add vmemmap and switch to buddy page allocator	pkvm: x86: Add vmemmap and switch to buddy page allocator
2611017650f194c536ea17bbcc5f358b9441fd60	pkvm: x86: Introduce host EPT pgtable support	pkvm: x86: Introduce host EPT pgtable support
d057cc4d3956e3df5e3f18927c125c61e1d47fb5	pkvm: x86: Create host EPT pgtable in init-finalise hypercall	pkvm: x86: Create host EPT pgtable in init-finalise hypercall
41b823697096ae1958b50d1a6c9a0b675ca808b8	pkvm: x86: Add pgtable API pkvm_pgtable_lookup	pkvm: x86: Add pgtable API pkvm_pgtable_lookup
e19b4a0009af8215c1a68f980bad66d65bab6ae0	pkvm: x86: Introduce find_mem_range API	pkvm: x86: Introduce find_mem_range API
10d2d5d99157f091d05cebd2b4deb0796fed2216	pkvm: x86: Dynamically handle host MMIO EPT violation	pkvm: x86: Dynamically handle host MMIO EPT violation
face2d7fb814b402649e44875022423862a72442	Misc	Misc
d5ce74dc157f0fb19eef9e95e232dbb9f7957fb1	pkvm: x86: Enable VPID for host VM	pkvm: x86: Enable VPID for host VM
ab3de368f50fd39ec23e378539114167b9a7cfa2	pkvm: x86: Add pKVM debug support	pkvm: x86: Add pKVM debug support
4d11b8b15c940b9d2fab1c193dde04504fe871be	pkvm: x86: Support get_pcpu_id	pkvm: x86: Support get_pcpu_id
71619a51e770ebe6fd704f53e2011b68ad131c42	pkvm: x86: Handle pending nmi in pKVM runtime	pkvm: x86: Handle pending nmi in pKVM runtime
6c3e04d07e93cbbaf915ca32ff707a34bb780524	VMX emulation	VMX emulation
744165571b2fad56db91e690b43ace90f2d1850c	pkvm: x86: Add memcpy lib	pkvm: x86: Add memcpy lib
e1cc0dc76aaccaa9424030b7ae665344814d34c3	pkvm: x86: Add memory operation APIs for for host VM	pkvm: x86: Add memory operation APIs for for host VM
538620eec714ac26c4c463c832d5628fd4c74977	pkvm: x86: Do guest address translation per page granularity	pkvm: x86: Do guest address translation per page granularity
ea75c0c4bfe5af3250a537c2b631794cfe0ef031	pkvm: x86: Add check for guest address translation	pkvm: x86: Add check for guest address translation
854828a784efc9337ad5462c2dee78916d854852	pkvm: x86: Add hypercalls for shadow_vm/vcpu init & teardown	pkvm: x86: Add hypercalls for shadow_vm/vcpu init & teardown

1. 加入intel pkvm文档和`CONFIG_PKVM_INTEL`
2. 准备把host linux降权成一个vm：给每个cpu准备pcpu/host vcpu结构
3. 把pKVM runtime做成独立binary，并加入reserved memory、allocator、MMU page table、host EPT、VPID、debug/NMI等基础设施

71be221646fe — pkvm: x86: Introduce CONFIG_PKVM_INTEL
d5b286f5a4e4 — pkvm: x86: Introduce pkvm_host_deprivilege_cpus
1e23286631a1 — pkvm: x86: Allocate vmcs and msr bitmap pages for host vcpu
304cecd867f7 — pkvm: x86: Add vmexit handler for host vcpu
3282557218ed — pkvm: x86: Build pKVM runtime as an independent binary
facc2107e6f8 — pkvm: x86: Deprivilege host OS
11a21ef399dd — Isolate pKVM & host
d057cc4d3956 — pkvm: x86: Create host EPT pgtable in init-finalise hypercall


## VMX 模拟、EPT 模拟、page-state 内存保护起步

5b9c53077cac1fbcb0c9be1c087e6296387395df	KVM: VMX: Add new kvm_x86_ops vm_free	KVM: VMX: Add new kvm_x86_ops vm_free
955255c36b6bdcbc882025553c6784cd074080a2	KVM: VMX: Add initialization/teardown for shadow vm/vcpu	KVM: VMX: Add initialization/teardown for shadow vm/vcpu
f2f58eb06913cd06a567d468e53ddac3a8c46661	pkvm: x86: Add hash table mapping for shadow vcpu based on vmcs12_pa	pkvm: x86: Add hash table mapping for shadow vcpu based on vmcs12_pa
e88ebf2679b3baaae4a1b1b7322779dcb53941a9	pkvm: x86: Add VMXON/VMXOFF emulation	pkvm: x86: Add VMXON/VMXOFF emulation
a51a7ab519608d78407857a2d13ec1d1f3cebd19	pkvm: x86: Add has_vmcs_field() API for physical vmx capability check	pkvm: x86: Add has_vmcs_field() API for physical vmx capability check
25b1a2f5e244217250cece142dd782a5869eae6e	KVM: VMX: Add more vmcs and vmcs12 fields definition	KVM: VMX: Add more vmcs and vmcs12 fields definition
702dcf57d395e9d598987eb6d54ed6573aaf3469	pkvm: x86: Init vmcs read/write bitmap for vmcs emulation	pkvm: x86: Init vmcs read/write bitmap for vmcs emulation
b01b51553ac34b84c17cd41d97667a852ed4b86a	pkvm: x86: Initialize emulated fields for vmcs emulation	pkvm: x86: Initialize emulated fields for vmcs emulation
82aa50ce4ff6a810f10b5346b0758e555e361350	pkvm: x86: Add msr ops for pKVM hypervisor	pkvm: x86: Add msr ops for pKVM hypervisor
cdb61004ecabf974ae4e6806b8800dadb6e5f8fd	pkvm: x86: Move _init_host_state_area to pKVM hypervisor	pkvm: x86: Move _init_host_state_area to pKVM hypervisor
e81139d9417f5698a9004885203db0a8f88cd040	pkvm: x86: Add vmcs_load/clear_track APIs	pkvm: x86: Add vmcs_load/clear_track APIs
2122a8c4a0167de92180ef7e8437aaff83fc164a	pkvm: x86: Add VMPTRLD/VMCLEAR emulation	pkvm: x86: Add VMPTRLD/VMCLEAR emulation
eb9058d37a2fa05f457392c80e6b67d0eb5e6383	pkvm: x86: Add VMREAD/VMWRITE emulation	pkvm: x86: Add VMREAD/VMWRITE emulation
29ae5b0f525e5acadbf6bdba64cdca138eb05b29	pkvm: x86: Add VMLAUNCH/VMRESUME emulation	pkvm: x86: Add VMLAUNCH/VMRESUME emulation
9b59db40042b1c5ea671064592b74093c3c1c14b	pkvm: x86: Add INVEPT/INVVPID emulation	pkvm: x86: Add INVEPT/INVVPID emulation
5cde477b7ba23de65e63add7dfc2d9fb2ba58478	pkvm: x86: Initialize msr_bitmap for vmsr	pkvm: x86: Initialize msr_bitmap for vmsr
6adb807bea718c6b614d08d7bb5a1d9b9cdf8570	pkvm: x86: Add vmx msr emulation	pkvm: x86: Add vmx msr emulation
3d43252fcdf79b2bd801b4ff5f8e93a3b66604c6	EPT emulation	EPT emulation
7709d004c8b6e314a52d23d121fa8a15e5179b98	pkvm: x86: Pre-define the maximum number of supported VMs	pkvm: x86: Pre-define the maximum number of supported VMs
bc4c43c803ac96cf5659f6cb3d54cfee9fcf1feb	pkvm: x86: init: Reserve memory for shadow EPT	pkvm: x86: init: Reserve memory for shadow EPT
88e429f9b878f0ab3aee67a8223dd3493692806c	pkvm: x86: Initialize the shadow EPT pool	pkvm: x86: Initialize the shadow EPT pool
76011ef693d053033257ebef8b1984e66fd410f1	pkvm: x86: Introduce shadow EPT	pkvm: x86: Introduce shadow EPT
d351daac74100618e5bd2661ef76f75c8ee6bda4	pkvm: x86: Introduce vEPT to record guest EPT information	pkvm: x86: Introduce vEPT to record guest EPT information
b483c612bd29f05e568ccf92c44206e76c67e71b	pkvm: x86: Add API to get the max phys address bits	pkvm: x86: Add API to get the max phys address bits
50a9f4aebcdeb6708fdc39c1e7e6c0e8d9dded25	pkvm: x86: Initialize ept_zero_check	pkvm: x86: Initialize ept_zero_check
cf2d0460b8beb1a825bb00f7d173c62db0ccb5b0	pkvm: x86: Add support for pKVM to handle the nested EPT violation	pkvm: x86: Add support for pKVM to handle the nested EPT violation
8cfde1a02335ec0439aa75cd431a7742f3f9def7	pkvm: x86: Introduce PKVM_ASSERT	pkvm: x86: Introduce PKVM_ASSERT
60c2ce9dc9eeb011a5f553e0fda63a7c6e3833b3	pkvm: x86: add pkvm_pgtable_unmap_safe for a safe unmap	pkvm: x86: add pkvm_pgtable_unmap_safe for a safe unmap
63bdfde237aca276a15909b52aebe5142ff2c74a	pkvm: x86: Introduce shadow EPT invalidation support	pkvm: x86: Introduce shadow EPT invalidation support
6d8c5dc3d79d4743c5b5115a92f4e12de16031b3	pkvm: x86: Add INVEPT instruction emulation	pkvm: x86: Add INVEPT instruction emulation
ede76e6113d4a182d95cccbbdabc402d270bcd56	pkvm: x86: Switch to use shadow EPT for nested guests	pkvm: x86: Switch to use shadow EPT for nested guests
c19bb3527ce6fa08509c83a28a504e777235875b	Memory protection based on page state	Memory protection based on page state
686cd048684912c526acec736c90796c9d8c7082	pkvm: x86: Introduce pkvm_pgtable_annotate	pkvm: x86: Introduce pkvm_pgtable_annotate
1e1daab911d8c14f0cd6fc28d2b6c0254f136029	pkvm: x86: Use host EPT to track page ownership	pkvm: x86: Use host EPT to track page ownership
848dc563249ba822aee97ae255ab850ddbbb046b	pkvm: x86: Use SW bits to track page state	pkvm: x86: Use SW bits to track page state
d8eb32ba30169902b3e34ae752c8593a7f50792b	pkvm: x86: Add the record of the page state into page table entry	pkvm: x86: Add the record of the page state into page table entry
2714413ff0a489a6f4c24d3348dd51c937b547bd	pkvm: x86: Expose host EPT lock	pkvm: x86: Expose host EPT lock
3740832bbd9de3f7a8cfbf47aefbf1dec8ae6024	pkvm: x86: Implement do_donate() helper for donating memory	pkvm: x86: Implement do_donate() helper for donating memory
1fd8bbac18c11270e9a26d7a53ce35da64dd397b	pkvm: x86: Implement __pkvm_hyp_donate_host()	pkvm: x86: Implement __pkvm_hyp_donate_host()
124fa2a67305a884c4effb8ca2f9dfe29cc17788	pkvm: x86: Donate shadow vm & vcpu pages to hypervisor	pkvm: x86: Donate shadow vm & vcpu pages to hypervisor
f2511189b0f83bbb3169f54adab75b97983da31b	pkvm: x86: Implement do_share() helper for sharing memory	pkvm: x86: Implement do_share() helper for sharing memory
2db69a35ee44206706453221bc2cdb5725c61f00	pkvm: x86: Implement do_unshare() helper for unsharing memory	pkvm: x86: Implement do_unshare() helper for unsharing memory
6ecaf7582046165f988f5224998eccd87bc1f35e	pkvm: x86: Add pgtable override helper functions for map/unmap/free leaf	pkvm: x86: Add pgtable override helper functions for map/unmap/free leaf
8b8b9dd0c85fa6190632668284cf9e06a395a0fd	pkvm: x86: Use page state API in shadow EPT for normal VM	pkvm: x86: Use page state API in shadow EPT for normal VM
54beb98682fbb40d5ed2627e9ed7903184bb1830	pkvm: x86: implement __pkvm_host_donate_guest()	pkvm: x86: implement __pkvm_host_donate_guest()
e9d021cf92564d4172f06ef3e63324cb699c7de7	pkvm: x86: implement __pkvm_host_undonate_guest()	pkvm: x86: implement __pkvm_host_undonate_guest()
8447dc601f9dde3fadc8a13e417ca9dfcabc5868	REVERTME: KVM: x86: Introduce vm_type to differentiate default VMs from protected VMs.	REVERTME: KVM: x86: Introduce vm_type to differentiate default VMs from protected VMs.
5afd02899447ad98a0fc3577c386634bdf8ee477	REVERTME: HACK: kvm: pin all the pages used by protected VM	REVERTME: HACK: kvm: pin all the pages used by protected VM
90defcf665e557b0da6dbfc0970ede0502e86583	pkvm: x86: add vm_type for shadow_vm	pkvm: x86: add vm_type for shadow_vm
ba8f27a94a4ba86c8aa42bb3643230fe0a25135b	pkvm: x86: add pgt_entry_mapped callback for pgtable ops	pkvm: x86: add pgt_entry_mapped callback for pgtable ops
5cbc19b43eba908e2b0c64e66c3a993b054f059e	pkvm: x86: add shadow EPT invalidation & destroy support for protected VM	pkvm: x86: add shadow EPT invalidation & destroy support for protected VM
15cf8b9753ddf95dddfa70107cb48fc5316a13a9	pkvm: x86: donate page in shadow EPT violation handler for protected VM	pkvm: x86: donate page in shadow EPT violation handler for protected VM
b67a6f859bbc227580e589f3a42d675b1d61e272	pkvm: x86: support initializing shadow_vm for protected VM	pkvm: x86: support initializing shadow_vm for protected VM
2fdbe553f1c6c6fa29de2abda9a7ab4303d71242	pkvm: x86: remap shadow EPT with new prot in EPT violation handling	pkvm: x86: remap shadow EPT with new prot in EPT violation handling
a0c65a85d4f8e901181b53317b9ed65c7e3c600c	DEBUG: pkvm: x86: add vmexit trace support	DEBUG: pkvm: x86: add vmexit trace support
9335347092da3078b1a57d99846459f87216e40a	pkvm: x86: implement __pkvm_guest_share_host()	pkvm: x86: implement __pkvm_guest_share_host()
a359068ebf5db41a11360f7b41ed99ff9371045f	pkvm: x86: implement __pkvm_guest_unshare_host()	pkvm: x86: implement __pkvm_guest_unshare_host()
91667bc925cf77897b171c1b737fbdbf4de00c9f	pkvm: x86: check multiple page state when host_undonate_guest()	pkvm: x86: check multiple page state when host_undonate_guest()
c4741e090f6c6edb8ee54416f4eda84b007ac327	pkvm: x86: add share/unshare memory hypercalls and handler	pkvm: x86: add share/unshare memory hypercalls and handler
dc74d8eefc57b816aa75727c896349d13e6027ee	pkvm: x86: add io virtual address space	pkvm: x86: add io virtual address space
98afc07468274f15eac2beb4424a5d20507764ad	pkvm: x86: add PKVM_PAGE_IO_NOCACHE	pkvm: x86: add PKVM_PAGE_IO_NOCACHE

1. 先让host kvm还能像以前一样发起vmx操作：pkvm开始模拟vmxon/vmxoff、vmread/vmwrite、vmlaunch/vmresume这些路径。
2. 然后开始接管ept：pkvm建shadow ept，用它记录和控制guest最后能访问哪些物理页。
3. 再补上page-state：记录一页属于host、guest、pkvm还是shared，并实现donate/share/unshare这些基础动作。

关键commit

e88ebf2679b3 pkvm: x86: add vmxon/vmxoff emulation
eb9058d37a2f pkvm: x86: add vmread/vmwrite emulation
29ae5b0f525e pkvm: x86: add vmlaunch/vmresume emulation
76011ef693d0 pkvm: x86: introduce shadow ept
cf2d0460b8be pkvm: x86: add support for pkvm to handle the nested ept violation
c19bb3527ce6 memory protection based on page state
1e1daab911d8 pkvm: x86: use host ept to track page ownership
3740832bbd9d pkvm: x86: implement do_donate() helper for donating memory

9070f520197aa36eb5667ec9efe08af966815922	pkvm: x86: add local apic support	pkvm: x86: add local apic support
8e5e8047fd994d2effe7841b79fa8c5f198dbe07	pkvm: x86: add spin lock to protect MMU map/unmap	pkvm: x86: add spin lock to protect MMU map/unmap
92f045edb09314a0c0b894d6194fb6e52e64fe28	pkvm: x86: emulate IA32_APIC_BASE msr	pkvm: x86: emulate IA32_APIC_BASE msr
74bde4b746e05eed986513edd56683bb4deb6826	pkvm: x86: intercept the APIC_ID msr	pkvm: x86: intercept the APIC_ID msr
68ec0e8ed842b9c543171ebb27f75b84617b5a4e	pkvm: x86: introduce vcpu kick helper function	pkvm: x86: introduce vcpu kick helper function
6a288c18f1433a7988c6d9e3dbe3089940d47bd2	pkvm: x86: intrdoucing IN/OUTSIDE_GUEST_MODE vcpu mode	pkvm: x86: intrdoucing IN/OUTSIDE_GUEST_MODE vcpu mode
17f65fc63641b5ddd76e4167d3112061f36153b7	pkvm: x86: add support to do tlb shootdown for primary VM	pkvm: x86: add support to do tlb shootdown for primary VM
52c05e86aeb96e3f68fc1b1478d86c3d00c6c246	pkvm: x86: add pkvm_pgtable as flush_tlb parameter	pkvm: x86: add pkvm_pgtable as flush_tlb parameter
9f30878e1d643109dbd2bfd3e87f1359d84d2d5b	pkvm: x86: add support to do tlb shootdown for nested VM	pkvm: x86: add support to do tlb shootdown for nested VM
3e6ac60dd36cbef15c0997c0de5d70e77c2bfb09	KVM: vmx: add tlb_remoe_flush/_with_range	KVM: vmx: add tlb_remoe_flush/_with_range
aac8fccca0642ef53c7c2d42f75ef7365b6d0887	pkvm: x86: implement for tlb_remote_flush/_with_range	pkvm: x86: implement for tlb_remote_flush/_with_range
15a7b856a44b6b97b29e5454911be448c323ae58	pkvm: x86: simplify invept emulation	pkvm: x86: simplify invept emulation
26c3599f9e7f37bdcc0069c2a7a3d5562b2ed025	pkvm: x86: initialize IOMMU cap before deprivilege	pkvm: x86: initialize IOMMU cap before deprivilege
3e207484958df5c5edbb7ffdfe748e3aa4dd7c70	pkvm: x86: add initial function to finalise IOMMU	pkvm: x86: add initial function to finalise IOMMU
eb08461dfe0cadebe55d62be947fb44125d67e29	pkvm: x86: init: add percpu pkvm_enabled flag	pkvm: x86: init: add percpu pkvm_enabled flag
d1e7cd11e8c303f1a9831c6313a80a80b065eef4	pkvm: x86: init: isolate IOMMU from host VM	pkvm: x86: init: isolate IOMMU from host VM
942381e22a6f3a139167b23781a3eef1a93ad663	pkvm: x86: init: check if pkvm IOMMU can support all the enumerated PCI devices	pkvm: x86: init: check if pkvm IOMMU can support all the enumerated PCI devices
ec9e80a5d48bd7bc76ce7c0e107caf0ad55cad9d	pkvm: x86: default enable IOMMU scalable mode	pkvm: x86: default enable IOMMU scalable mode
4227e0d6c0caa812a918dfbf387ff4b803a2c99f	pkvm: x86: init: reserve memory for IOMMU	pkvm: x86: init: reserve memory for IOMMU
9fcb4e47a3dd05620c41e74823a24fb9b07bff68	pkvm: x86: add function for flushing CPU cache	pkvm: x86: add function for flushing CPU cache
e0a25193883f147bd6a643419815bcc2ae73b158	pkvm: x86: not to handle EPT violation for GPA overlapped with IOMMU	pkvm: x86: not to handle EPT violation for GPA overlapped with IOMMU
e69b7ec5075f7ddeb8c5ebacd641bc2cc85cc579	pkvm: x86: initialize the IOMMU pgt	pkvm: x86: initialize the IOMMU pgt
f8a8c273d08f878aa6eec2390436350e200dfa3e	pkvm: x86: extend pgtable_walk to support IOMMU page table	pkvm: x86: extend pgtable_walk to support IOMMU page table
03644b199d72e3b4f5ce2ecb414375262bad44cc	pkvm: x86: shadow IOMMU page table	pkvm: x86: shadow IOMMU page table
cce3952e738aee5e8f9a1497221ac412cafa0bc8	pkvm: x86: add page cache flushing for IOMMU w/o Page-Walk Coherency	pkvm: x86: add page cache flushing for IOMMU w/o Page-Walk Coherency
94382393d8a4e8a0955cdb619c036625c8a89d89	pkvm: x86: update pgtable to allow page walk cache flush	pkvm: x86: update pgtable to allow page walk cache flush
ba52a2600a312a794b01428be08f4a8b7a99ba2c	pkvm: x86: do paging structure cache flushing when map/unmap EPT	pkvm: x86: do paging structure cache flushing when map/unmap EPT
7c1ca2a29045d8ae88c5cfafe9617f912669474d	DEBUG: pkvm: x86: add debug to dump IOMMU shadow page table	DEBUG: pkvm: x86: add debug to dump IOMMU shadow page table
1f83dc5e743fee066058cc39501574bdab1b1bdb	pkvm: x86: save the IOMMU gcmd enabled features	pkvm: x86: save the IOMMU gcmd enabled features
8da75484048469c3429139edb9f5f13a1e0077b9	pkvm: x86: reserve memory for IOMMU Queued Invalidation	pkvm: x86: reserve memory for IOMMU Queued Invalidation
8d24798e52be22e525f18b6542074dcf9ddb434e	pkvm: x86: add IOMMU queue invalidation support	pkvm: x86: add IOMMU queue invalidation support
8ac5bbc76165338a387e7376ab8a3548e6d87abf	pkvm: x86: switch to the IOMMU root table maitained by pkvm	pkvm: x86: switch to the IOMMU root table maitained by pkvm
a06637bf70bf4a17abe469545f47ea51102dad77	pkvm: x86: make sure IOMMU DMA translation is enabled	pkvm: x86: make sure IOMMU DMA translation is enabled
e4a256ddf8c623410d5f0bc8bfe743cec3e13975	pkvm: x86: handle QI requests from host VM IOMMU driver	pkvm: x86: handle QI requests from host VM IOMMU driver
f641c78bbc32dd95f597a22fbd065ffa8b1f2bee	pkvm: x86: Disable IOMMU Device TLB capability for security.	pkvm: x86: Disable IOMMU Device TLB capability for security.
65097932e5ee8e498f1296f1b2a3feac8ebd2b36	pkvm: x86: intercept IOMMU MMIO GCMD/GSTS/CAP/ECAP/RTADD accessing from host VM	pkvm: x86: intercept IOMMU MMIO GCMD/GSTS/CAP/ECAP/RTADD accessing from host VM
c9383f0b9b73b2c09a8a202a34596434bb48da52	pkvm: x86: activate IOMMU	pkvm: x86: activate IOMMU
a1ec06ebd6df998d1c442563977cb15b33f2e047	REVERTME: DEBUG: pkvm: x86: bypass legacy IOMMU	REVERTME: DEBUG: pkvm: x86: bypass legacy IOMMU
9eb9a86d10887f05afb6e1b794c2e8a68b5273e6	pkvm: x86: Move pkvm guest hypercalls to uapi directory	pkvm: x86: Move pkvm guest hypercalls to uapi directory
03f058261ecdff2f05f585bf0c2b79953098f175	pkvm: x86: Add the handler for hypercalls of PV MMIO operations	pkvm: x86: Add the handler for hypercalls of PV MMIO operations
2f41ed618db7de43ae7279e4e60e8095ce6d19bf	pkvm: x86: Fix the vaddr parameter for flushing shadow EPT tlb	pkvm: x86: Fix the vaddr parameter for flushing shadow EPT tlb
0fdc8d73766d8a47de7d299245fa7d46d226c2ef	pkvm: x86: move EPT capability APIs to capabilities.h	pkvm: x86: move EPT capability APIs to capabilities.h
cdf59b9f616811248f08fbe0499fbfccaba544f3	pkvm: x86: add missing VMX capability checks before enabling VPID	pkvm: x86: add missing VMX capability checks before enabling VPID
4c7ec9f0de8cf23384116bfabe4979bb50cce551	pkvm: x86: add support to emulate invvpid instruction	pkvm: x86: add support to emulate invvpid instruction
928345cf327b73eebc9702f69e780ed20457c930	pkvm: x86: Fixes uninitialized Variable in __pkvm_guest_share_host()	pkvm: x86: Fixes uninitialized Variable in __pkvm_guest_share_host()
077d04c66d34282d61cd393d8e5f5fa23b8376fc	pkvm: x86: don't split a huge page when invalidating a SEPT entry	pkvm: x86: don't split a huge page when invalidating a SEPT entry
387094db706364a061e3a15ab0bd0c2df41dc40a	pkvm: x86: add pgtable_free_child() to remove duplicated code	pkvm: x86: add pgtable_free_child() to remove duplicated code
fe40fc232b267315f08305729b12cff501949dcd	pkvm: x86: flush remote TLBs with range in mmu_notifier callbacks	pkvm: x86: flush remote TLBs with range in mmu_notifier callbacks
6e83753db5321a43838951f285cdf1e115b1132a	KVM: stats: Add VM stat for remote tlb flush with range	KVM: stats: Add VM stat for remote tlb flush with range
8b715c73a5e315cf5eb180df8f8692bf16439394	pkvm: x86: Rename scalable-mode shadow table ops	pkvm: x86: Rename scalable-mode shadow table ops
7f7fd88a52e3d20990e31cd0d7d15d2ee73fc05d	pkvm: x86: Introduce iommu_lm_id_ops	pkvm: x86: Introduce iommu_lm_id_ops
92c412c0fb1d01b8e252e5e1591c36daae15cdb9	pkvm: x86: Extend sync_shadow_id to cover legacy mode	pkvm: x86: Extend sync_shadow_id to cover legacy mode
54710925428facec9f92ab18a0982f3631f3b69d	pkvm: x86: Extend context_cache_invalidate to cover legacy mode	pkvm: x86: Extend context_cache_invalidate to cover legacy mode
222c5227c6204e16b3dad951ef95c8da3799dc28	pkvm: x86: Extend gcmd_srtp & gcmd_te to cover legacy mode	pkvm: x86: Extend gcmd_srtp & gcmd_te to cover legacy mode
75245789f38ce0b73a7a3407310286395d184993	pkvm: x86: Enable mode selecting automatically	pkvm: x86: Enable mode selecting automatically
906d64d4ccd6927d10d06c7c95496867dcd8bd5a	Revert "REVERTME: DEBUG: pkvm: x86: bypass legacy IOMMU"	Revert "REVERTME: DEBUG: pkvm: x86: bypass legacy IOMMU"
c908a610377f6e3ee584a76367e9b1cf8d108a82	pkvm: x86: hide EPT A/D support from pKVM high	pkvm: x86: hide EPT A/D support from pKVM high
18d8e5aa554cc439a145843800b15e75567a4f0a	pkvm: x86: Pass iommu ecap value to sync_data	pkvm: x86: Pass iommu ecap value to sync_data
e1673da78536e9152fb1141e9dce7a1a579fa436	pkvm: x86: Set the vcpu->mmio_is_write when handle MMIO operation	pkvm: x86: Set the vcpu->mmio_is_write when handle MMIO operation
f6f003b6819c78c29ed6ba02e13e60941fd74c7f	pkvm: x86: Remove pgtable_pte_is_counted	pkvm: x86: Remove pgtable_pte_is_counted
4351f4d14d1488d6248175ad4d123d9cc0c3b4ad	pkvm: x86: Initialize every SEPT entry with "suppress #VE" bit set	pkvm: x86: Initialize every SEPT entry with "suppress #VE" bit set
140fab7d89959d6a6b79a9780388b9b4d1b3db0b	pkvm: x86: Enable EPT_VIOLATION_VE in pkvm by default	pkvm: x86: Enable EPT_VIOLATION_VE in pkvm by default
56a4ed35090862d7b983f555dfbd1a780f071c94	pkvm: x86: Enable Virtualization Exception Information	pkvm: x86: Enable Virtualization Exception Information
4369097d35153cb39441b99bc6ee4b66b681e453	pkvm: x86: Add support for clear "Suppress #VE" bit in pkvm	pkvm: x86: Add support for clear "Suppress #VE" bit in pkvm
34e8432eb45712398c1ed27a8e1487197d35789b	KVM: x86: Using SET_MMIO_VE hypercall to clear "Suppress #VE" bit	KVM: x86: Using SET_MMIO_VE hypercall to clear "Suppress #VE" bit
fe569c87d534e8f4458543f6b4d46944264b9951	pkvm: x86: Fix some minor issues in macros	pkvm: x86: Fix some minor issues in macros
79fb4e01c3323b9333248fa8935c407e55c9114f	pkvm: x86: Introduce pgstate_pgt for shadow VM	pkvm: x86: Introduce pgstate_pgt for shadow VM
253a7b4f942805e0b9ab4afcea40f8f37384ef27	pkvm: x86: Move page state management from shadow ept to pgstate_pgt	pkvm: x86: Move page state management from shadow ept to pgstate_pgt
d44e9a529eb6e352c3f59d0421db33f8b5228d17	pkvm: x86: Add new hypercall for KVM high to share passthrough dev info	pkvm: x86: Add new hypercall for KVM high to share passthrough dev info
756f4fbc7da3b7667560e667fc56831d1592801a	pkvm: x86: Introduce pkvm_ptdev structure	pkvm: x86: Introduce pkvm_ptdev structure
ca911169d1ed74cb2c863d39db963d7dc92db5fc	pkvm: x86: Implement the ADD_PTDEV hypercall	pkvm: x86: Implement the ADD_PTDEV hypercall
a019a98e6eab1ce418900589cc569d3edaff8d04	pkvm: x86: Sync IOMMU page table for the attached ptdev	pkvm: x86: Sync IOMMU page table for the attached ptdev
57436548b63c2ece4ce8ae14f518cd08e1e6af3a	pkvm: x86: Audit ptdev did	pkvm: x86: Audit ptdev did
870e810dc1582d7062c1cdadf1f081a5f8832d7e	pkvm: x86: Introduce pgtable sync map function	pkvm: x86: Introduce pgtable sync map function
83587b2745b0de793c9be761daea8a760e51c5b5	pkvm: x86: Pre-populate pgstate pgt if protected VM has passthrough device	pkvm: x86: Pre-populate pgstate pgt if protected VM has passthrough device
2c55933003a14fc2aea51b7cd1b09a5421336d36	pkvm: x86: Add pgt_override pointer in pkvm_mem_trans_desc for host	pkvm: x86: Add pgt_override pointer in pkvm_mem_trans_desc for host
33c05fcc724a8f5db5166b3caad9bc69cf8a8c9a	pkvm: x86: Add fastpath to donate a page from host to guest	pkvm: x86: Add fastpath to donate a page from host to guest
13c05ce881d31fc8606da34f34d7d9b833eae425	pkvm: x86: Fix inline asm in pkvm_init_ept_page()	pkvm: x86: Fix inline asm in pkvm_init_ept_page()
ce8544b7b2193b25e00edba466bce5aeac5a8d9f	pkvm: x86: Fix NULL deref in sync_shadow_context_entry()	pkvm: x86: Fix NULL deref in sync_shadow_context_entry()
fb88781b80eb98ad1127a6e988cadb0063137cc0	pkvm: x86: Memory protection with legacy IOMMU in pass-through mode	pkvm: x86: Memory protection with legacy IOMMU in pass-through mode
fe163fa5213e5c9bf38d648d4812d9cec50176be	pkvm: x86: Rename context_clear_dte() to context_sm_clear_dte()	pkvm: x86: Rename context_clear_dte() to context_sm_clear_dte()
b96768f503f4181e89e420dc60a6ae5da0fc407c	pkvm: x86: Fix a minor calculating issue	pkvm: x86: Fix a minor calculating issue
8fe5bafc782c07f3c3df7451ffdd0a42869866c1	pkvm: x86: Add helper function shadow_vcpu_is_protected()	pkvm: x86: Add helper function shadow_vcpu_is_protected()
204cb528183965ead7386f7c70a9d8d3c139ec1c	pkvm: x86: Add support of cpuid for protected VM in pkvm	pkvm: x86: Add support of cpuid for protected VM in pkvm
b65e60c8a99b6b3d6ecf9b89f7b8b2c3a5b5318f	REVERTME: kvm: not to pin the non-memory pfn	REVERTME: kvm: not to pin the non-memory pfn
5e111cb61f600ea8d2dcec3f483353d092a9c7ec	REVERTME: kvm: fix up non-working pinning of protected VM pages	REVERTME: kvm: fix up non-working pinning of protected VM pages
8e345f5c3ff1538934413b4231856c723c838917	pkvm: x86: Not to memset for non-memory phys	pkvm: x86: Not to memset for non-memory phys
be4851e0901306d3b08e5efc47eaedc057721fd4	pkvm: x86: Fix detection of IOMMU coherency in legacy mode	pkvm: x86: Fix detection of IOMMU coherency in legacy mode
d89d7013692b5510d7b486909d1d666b8d30fd6d	pvkm: x86: Fix misplaced comment for nested IOMMU translation check	pvkm: x86: Fix misplaced comment for nested IOMMU translation check
cfb72e3555da28082b024ee957a6e53b1440f7ce	pkvm: x86: Fix deadlocks caused by __pkvm_guest_share_host()/__pkvm_guest_unshare_host()	pkvm: x86: Fix deadlocks caused by __pkvm_guest_share_host()/__pkvm_guest_unshare_host()
243b2e2aa5848449de1cdf2510541b9b661ec49d	pkvm: x86: Fix missed TLB flushing in pgtable_unmap_leaf	pkvm: x86: Fix missed TLB flushing in pgtable_unmap_leaf
d1974f600f2f1db789a55c7c0f6be9a8fe50afdb	pkvm: x86: Adjust legacy IOMMU mode to use ptdev framework	pkvm: x86: Adjust legacy IOMMU mode to use ptdev framework
444283a7d4cd9d79ab5c24cd0f155cb8728b2876	pkvm: x86: Optimize domain-selective context cache sync in legacy mode	pkvm: x86: Optimize domain-selective context cache sync in legacy mode
ec99ca964a2837d4b627696eef0998a4b3822c9d	pkvm: x86: Support attaching ptdev to protected VM in legacy mode	pkvm: x86: Support attaching ptdev to protected VM in legacy mode
98c718984ac63214271820088202f7c81def19c2	pkvm: x86: Rename for shadow EPT memory pool related functions	pkvm: x86: Rename for shadow EPT memory pool related functions
7b4320dc150eef28bcbcbe56181d22e30895fc33	pkvm: x86: Flush caches when allocating pgtable paging page	pkvm: x86: Flush caches when allocating pgtable paging page
087a4ebecd76746aae0b51b2fba785b5fd9461a1	pkvm: x86: Introduce iommu_coherency flag for ptdev	pkvm: x86: Introduce iommu_coherency flag for ptdev
92be6cd0f6958d4630f5943a3c70540deb9ddf1d	pkvm: x86: Setup pgstate_pgt coherency for ptdev	pkvm: x86: Setup pgstate_pgt coherency for ptdev
8c3b5e98ae67bcab6adf9df6ae3da020d4afb0ef	pkvm: x86: add addr and size parameter to flush_tlb callback	pkvm: x86: add addr and size parameter to flush_tlb callback
e263c4d724475906ed6639812071b32959d5aba5	pkvm: x86: create setup_iotlb_qi_desc helper function	pkvm: x86: create setup_iotlb_qi_desc helper function
955b2691285d75dc500635b27f80a8d8acd38bd2	pkvm: x86: Reserve more QI descriptor pages for each CPU	pkvm: x86: Reserve more QI descriptor pages for each CPU
90ba9ecfec6a57e993fa254fb2bdbe410e6d94a3	pkvm: x86: do iotlb flushing when flushing tlb for host EPT	pkvm: x86: do iotlb flushing when flushing tlb for host EPT
8cdfc708ed17e8eabe154eaf95ba0c5322486e90	pkvm: x86: Add explicit dependency on INTEL_IOMMU	pkvm: x86: Add explicit dependency on INTEL_IOMMU
b7edcbb16b290229f00adf4a5e47d219efb6b1e0	pkvm: x86: Reserve memory for shadow IOMMU page tables	pkvm: x86: Reserve memory for shadow IOMMU page tables
082a47acc054a12071f9c8056b23169d268a587d	pkvm: x86: Rename pgstate_pgt_mm_ops to shadow_sl_iommu_pgt_mm_ops	pkvm: x86: Rename pgstate_pgt_mm_ops to shadow_sl_iommu_pgt_mm_ops
9e3ec16a7ad07b6a4332dd4e011b99955775d801	pkvm: x86: Add get/put API for shadow IOMMU page tables	pkvm: x86: Add get/put API for shadow IOMMU page tables
5351eca0ae69976f5cddb781599d68ca27eaa1c1	pkvm: x86: Support shadowing ptdev's vIOMMU page tables	pkvm: x86: Support shadowing ptdev's vIOMMU page tables
ddbe7d209b0d15575d8b009d90f01d6b95c2babf	pkvm: x86: Improve pkvm_pgtable_sync_map() to support unmapping	pkvm: x86: Improve pkvm_pgtable_sync_map() to support unmapping
37cac11d4037235a4fb03d6e98ab35c6be6e8d81	pkvm: x86: Improve pkvm_pgtable_sync_map() documentation	pkvm: x86: Improve pkvm_pgtable_sync_map() documentation
dfbb18d7226a9febb95fca5d1134e051639d007f	pkvm: x86: Add pkvm_pgtable_sync_map_range()	pkvm: x86: Add pkvm_pgtable_sync_map_range()
6b0eff463fba7edc39ce2c5fb26dc9036a8b55ad	pkvm: x86: Rename iommu's "shadow pgt" to "shadow id"	pkvm: x86: Rename iommu's "shadow pgt" to "shadow id"
f04c4c9e366fcebbf7279d1518ebe96d8e69e074	pkvm: x86: Use shadow IOMMU page tables for legacy IOMMU	pkvm: x86: Use shadow IOMMU page tables for legacy IOMMU
ad67377ae2228cdcf5c12fb2fe0de3cbf2daf675	pkvm: enable I/O execution control for host vm	pkvm: enable I/O execution control for host vm
ecfbc417183f7a3e91836a8a5b508121f6705406	pkvm: introduce port I/O emulation framework	pkvm: introduce port I/O emulation framework
017eac68a45a560fb8144c2dfb233b915f36b36a	pkvm: support physical port I/O access	pkvm: support physical port I/O access
2b7c661fb3cd2d1a8ada10a0f4f3888a330bc153	pkvm: implement port I/O emulation	pkvm: implement port I/O emulation
797ba151f022298a8b7f95229f5ac0d1019bbbb9	pkvm: emulate PCI config space access	pkvm: emulate PCI config space access
d4c091b6cad6c96b828b1e5aff69bcf463405205	pkvm: introduce memory mapped I/O emulation framework	pkvm: introduce memory mapped I/O emulation framework
1bdf1bab564bd79e5964aa93e7c48c6b1b47d7a7	pkvm: support physical MMIO access	pkvm: support physical MMIO access
1a196bdbefb69b6eb9f6cafd2a639aa043c7b365	pkvm: implement memory mapped I/O emulation	pkvm: implement memory mapped I/O emulation
85e69275916055ac41adfe6aa36e719a51945dff	pkvm: fetch PCI mmconfig list from host	pkvm: fetch PCI mmconfig list from host
5143b3f6486796860ced01825a65d3a9b0ca8f99	pkvm: handle host PCI mmcfg access for ptdev	pkvm: handle host PCI mmcfg access for ptdev
239cd97d603107cdb65c1ca995ee996b2b44449b	pkvm: audit config space access for ptdev	pkvm: audit config space access for ptdev
c66462e27d4406203af2da17210fb2e4acf9c923	pkvm: x86: minor: add __init to some iommu init functions	pkvm: x86: minor: add __init to some iommu init functions
b2152894c6c418b545a323aabc3bc01e362efbfa	pkvm: x86: Fix missing newlines in prints	pkvm: x86: Fix missing newlines in prints
5e53b71b64a14c893e418e5e12ee79f1312a136f	pkvm: x86: Add ENDBR to exception handlers	pkvm: x86: Add ENDBR to exception handlers
6e2cb643d4187a9cb7e434205fd322d10a0c99c2	pkvm: x86: Fix compiling warnings	pkvm: x86: Fix compiling warnings
6593ee1f90a4ad6abe721c8fe7be699e8fb07408	pkvm: x86: Fix missing-prototypes compiling issue in pkvm_debugfs	pkvm: x86: Fix missing-prototypes compiling issue in pkvm_debugfs
b4ae8f177999092d8a2f216322f38052eafc7229	pkvm: x86: Fix compiling error when CONFIG_MITIGATION_SRSO=y	pkvm: x86: Fix compiling error when CONFIG_MITIGATION_SRSO=y
51e8f849a0c59b38e4f49ec2ac1c1f374a5d3cd3	pkvm: x86: Fix an issue with asm_goto_output usage	pkvm: x86: Fix an issue with asm_goto_output usage
b202fdd817533f5e6b071d3b4791dd4e269ce416	pkvm: x86: Fix compiling issues found by 0day for arch s390/powerpc	pkvm: x86: Fix compiling issues found by 0day for arch s390/powerpc
f2085cbc510f1dae530f93c37fd1e2dff0b5bd39	pkvm: x86: Fix compiling issue when CONFIG_HYPERV=n	pkvm: x86: Fix compiling issue when CONFIG_HYPERV=n
3ee30fe0c3e34b29ec6caf788efe8ef33e0a0686	pkvm: x86: Fix unitialized iotlb_lm_invalidate() return value	pkvm: x86: Fix unitialized iotlb_lm_invalidate() return value
7939f3835f3b36c3b4988ae747ad63e79b7da94d	pkvm: x86: Make iotlb_lm_invalidate() return value deterministic	pkvm: x86: Make iotlb_lm_invalidate() return value deterministic
135d768a42a506100607b88e21169d4cd152eeab	pkvm: x86: Fix deadlock between page donation and emulated IOTLB flush	pkvm: x86: Fix deadlock between page donation and emulated IOTLB flush
871915ada00b462a10d9e3a85490a2659b7f9587	pkvm: x86: Add kernel command line flag to enable pKVM	pkvm: x86: Add kernel command line flag to enable pKVM
b7768815bfe4ce6280c2740d0922c1ad28508627	pkvm: x86: Add check for unsupported vm_type	pkvm: x86: Add check for unsupported vm_type
2bebd3b9c6068bee2e11084d4702a8b0fda8f345	pkvm: x86: Add shadow_vm_is_protected() helper	pkvm: x86: Add shadow_vm_is_protected() helper
ad9645ed846bf4eecab2fcbda7f8e5882e3e0280	pkvm: x86: Add pkvm_is_protected_{vm,vcpu}() helpers for host	pkvm: x86: Add pkvm_is_protected_{vm,vcpu}() helpers for host
33eb6ef71e5a40dfc0b4e1f01cae750cd7b25	pkvm: x86: Rename KVM_X86_PROTECTED_VM to KVM_X86_PKVM_PROTECTED_VM	pkvm: x86: Rename KVM_X86_PROTECTED_VM to KVM_X86_PKVM_PROTECTED_VM
cfeba8e5cde26387d037d8aeb8143fce6c3aa3fc	pkvm: x86: Explicitly page-align ve_info	pkvm: x86: Explicitly page-align ve_info
cce761ad5c4f56a4539d3bf9c9a89cbd32c68e1b	pkvm: x86: De-hardcode vm_type size in __pkvm_init_shadow_vm()	pkvm: x86: De-hardcode vm_type size in __pkvm_init_shadow_vm()
5159ebe10118756f46319a99109ecc9d10658eaf	pkvm: x86: Parse pvmfw memory region info from the bootloader	pkvm: x86: Parse pvmfw memory region info from the bootloader
705577fe3e7342e5dd035d3b23fb7ef9729add32	pkvm: x86: Map pvmfw memory region for the hypervisor	pkvm: x86: Map pvmfw memory region for the hypervisor
c82d7911171446858a7b635ca930b9fc29af2b55	pkvm: x86: Protect pvmfw memory region from the host	pkvm: x86: Protect pvmfw memory region from the host
1da0371497725c31c4bd1a91f367a2fae1c54422	pkvm: x86: Introduce uAPI to set/query pvmfw info	pkvm: x86: Introduce uAPI to set/query pvmfw info
e4d7f995a8b8111f22cca8a4a99e469d520b013c	pkvm: x86: Add pkvm_hyp_types.h for pkvm_constants	pkvm: x86: Add pkvm_hyp_types.h for pkvm_constants
9de01881ed96300d83756304d5d3ef4f72f009de	pkvm: x86: Add pkvm_iommu_types.h for pkvm_constants	pkvm: x86: Add pkvm_iommu_types.h for pkvm_constants
be6802a642c5cd92dacf0d1887ba65a5aa4e90fd	pkvm: x86: Enforce pvmfw as the pVM entry point	pkvm: x86: Enforce pvmfw as the pVM entry point
1070072c9e1cb919d7dc93bfc2ff6465e669cae8	pkvm: x86: Copy pvmfw memory region into pVM memory	pkvm: x86: Copy pvmfw memory region into pVM memory
6e6963184f361782e71f4f776d2aa3787d709e28	pkvm: vmx: Enable EPT violation VE according to vmcs_config in pkvm_hyp	pkvm: vmx: Enable EPT violation VE according to vmcs_config in pkvm_hyp
af132acf50458d021e956f7178c3c3b67f383ac7	pkvm: retpoline: Fix objtool warning for naked reture	pkvm: retpoline: Fix objtool warning for naked reture
c05df00db50c9f020320460f5c5de021a7761712	pkvm: vmx: Refactor instance names for pkvm_host_vcpu and kvm_vcpu	pkvm: vmx: Refactor instance names for pkvm_host_vcpu and kvm_vcpu
c50f94481e10d47f39bbb3b2bc6c07a29f886cdf	pkvm: vmx: Not to protect pkvm text in debug mode	pkvm: vmx: Not to protect pkvm text in debug mode
fa0ee0bfff9fb3fcfb39bdfd5cee484073a4ed6c	pkvm: x86: Keep the __pkvm_ prefix for all pkvm symbols in debug mode	pkvm: x86: Keep the __pkvm_ prefix for all pkvm symbols in debug mode
e1edb4df3f142d816fa932bfd5ead25263704f88	pkvm: x86: Fix the reserved memory size for early alloc	pkvm: x86: Fix the reserved memory size for early alloc
49f841c77c13a132b6a4b03711bddef7c404556e	pkvm: x86: Allocate memory page for percpu	pkvm: x86: Allocate memory page for percpu
2a8a40b16e4d359a0298fdaeb06b4d28067464d6	pkvm: x86: Add .pkvm.data..percpu section	pkvm: x86: Add .pkvm.data..percpu section
c5f153302f2983273026bdabc3f8956d956bf8d3	HACK: pkvm: x86: Setup FS_BASE with the pkvm percpu offset	HACK: pkvm: x86: Setup FS_BASE with the pkvm percpu offset
50d0419491cccb4ba37b4217dc183bf62244ccae	pkvm: x86: Add pcpu_hot percpu data	pkvm: x86: Add pcpu_hot percpu data
fabcec48c1a6a2f74b55a2d9a1124c9ff303e0eb	pkvm: x86: Introduce def.h to control defines for pkvm hypervisor	pkvm: x86: Introduce def.h to control defines for pkvm hypervisor
4dd3f47ae8b13f69b3f577264d834a341138094a	pkvm: x86: Get cpu id via smp_processor_id	pkvm: x86: Get cpu id via smp_processor_id
182eec5269e5bf3e61a43c7d9fc7d393d2fa3d4b	pkvm: x86: Undefine printk kernel configs for the non-debug mode	pkvm: x86: Undefine printk kernel configs for the non-debug mode
c390a55c92ea611aa13486c2e61bb5189a4fa6c4	pkvm: io: Refactor the return value for try_emul_host_mmio	pkvm: io: Refactor the return value for try_emul_host_mmio
475cf5457d06e224373f2646e7dfe54416af3974	pkvm: io: Refactor the emulated mmio lookup	pkvm: io: Refactor the emulated mmio lookup
14face798a6d994fc39acf6e3f290df40ea98926	pkvm: io: Revise/fix to avoid potential mmio emulation issues	pkvm: io: Revise/fix to avoid potential mmio emulation issues
62156ae47b4f22bc00836a45f31560898c0792ed	pkvm: vmx: Improve VMWRITE emulating and eliminate VMCS copying	pkvm: vmx: Improve VMWRITE emulating and eliminate VMCS copying
f0f1fcdb22a8728f1f08e2a8918ced493e2c0bd0	pkvm: vmx: Refactor pkvm stack	pkvm: vmx: Refactor pkvm stack
6da6ce4e1d2ad0635552dcdae9b9e45c6f594364	pkvm: vmx: Split the main function handling vm-exit from pkvm_main	pkvm: vmx: Split the main function handling vm-exit from pkvm_main
189399f7afdf8c9ab0fa29f2ec95ee3545767252	pkvm: vmx: Add a new vm-exit handling process of pkvm	pkvm: vmx: Add a new vm-exit handling process of pkvm
fb2971c97ec51f36a834d2dbff27ea2f278cc19f	pkvm: vmx: Reshuffle the host CPU deprivilege process	pkvm: vmx: Reshuffle the host CPU deprivilege process
88632cc37f8cdccc00df76e24e8ff1d7bb2daba3	pkvm: vmx: Remove dead code of old vm-exit flow	pkvm: vmx: Remove dead code of old vm-exit flow
f1dc5119b293c3c7b03d35dd4591a283db90ce2d	kvm: pkvm: Add pkvm_high.c with KVM x86 ops	kvm: pkvm: Add pkvm_high.c with KVM x86 ops
5a2ab214a5025055fcd902f1d87f1efe4dac7c4b	kvm: vmx: Register x86 ops according to kernel configure	kvm: vmx: Register x86 ops according to kernel configure
a684e9d1df3a6c9046d0a63dc24dceda6a06136c	kvm: pkvm: Allow fallback to normal Linux/KVM host if pkvm_init fails	kvm: pkvm: Allow fallback to normal Linux/KVM host if pkvm_init fails
a653bf0d5c88410672fe87f9ab4593367201fa45	kvm: vmx: Use hyperv inline symbols for the pkvm hypervisor code	kvm: vmx: Use hyperv inline symbols for the pkvm hypervisor code
2ae03e60a6d1fc6f0a8ef730edd1bc831a5835d9	pkvm: vmx: Add vmcs config setup functionality	pkvm: vmx: Add vmcs config setup functionality
dd8acf24f5b508a8174e3f8383564d49056a43ae	pkvm: vmx: Setup the global vmcs cofig and capability	pkvm: vmx: Setup the global vmcs cofig and capability
25d7024110cee664d69c6ccc6bb8f2ba028d022f	pkvm: x86: Add boot_cpu_data symbole	pkvm: x86: Add boot_cpu_data symbole
d6bbdb5074d5014ad702c867425986ef10cc1cf1	pkvm: vmx: Reuse vmx instructions defined in vmx/vmx_ops.h	pkvm: vmx: Reuse vmx instructions defined in vmx/vmx_ops.h
1e3f8955f5f1d94f10bf562be0a844269095e9fe	pkvm: x86: Add kvm_x86_vendor_init	pkvm: x86: Add kvm_x86_vendor_init
264241894eb308d923527cb989864741d69a5e62	pkvm: vmx: Add vmx_hardware_setup	pkvm: vmx: Add vmx_hardware_setup
62a93f47e7011f3f6805442de316ffe6e51964d8	pkvm: vmx: Disable PMU support	pkvm: vmx: Disable PMU support
7b568fe3710f33c69936c6c45afef03848fa5838	pkvm: vmx: Disable the nested support	pkvm: vmx: Disable the nested support
8c4aac0a7c916e039c78d6da22ce259d84352396	pkvm: vmx: Disable SGX support	pkvm: vmx: Disable SGX support
cfda4efa5d037a5cd8abb37756efeebf7640f142	pkvm: vmx: Disable EPT AD bits feature	pkvm: vmx: Disable EPT AD bits feature
d32c1bf0421261cd0505062a2c47d860ef6a052c	kvm: pkvm: Prevent the host from using the VMX preemption timer	kvm: pkvm: Prevent the host from using the VMX preemption timer
3932a6e53a5dc6830b2c024df78765ac69e01401	kvm: pkvm: Disable the guest SMM support	kvm: pkvm: Disable the guest SMM support
2291dadc01f788c16b9dbe23517b6448fd373cbd	kvm: pkvm: Define kvm_call_pkvm helper	kvm: pkvm: Define kvm_call_pkvm helper
ca08fd93fe2dd8c077aea096a5eeb4fb8bcec3b4	pkvm: x86: Handle the KVM_CALL from the linux host	pkvm: x86: Handle the KVM_CALL from the linux host
a2f55dbdd9ca1c7029dbf0a4d63fe9a09de3a2f6	pkvm: x86: Add percpu kvm_vcpu pointer points to the host vcpu	pkvm: x86: Add percpu kvm_vcpu pointer points to the host vcpu
a831ac0fdc46f72fbbd9ca4778765be4a4fe0203	pkvm: x86: Add __cpu_possible_mask and nr_cpu_ids symbols	pkvm: x86: Add __cpu_possible_mask and nr_cpu_ids symbols
e2e85908004ed31e342d189b03eef78a607bf4ee	pkvm: vmx: Implement enable/disable_virtualization_cpu callbacks	pkvm: vmx: Implement enable/disable_virtualization_cpu callbacks
5a422ae3220fd7ba9619c28855348e8336aae81b	REVERTME: pkvm: vmx: Enable the emulate-based VMX support	REVERTME: pkvm: vmx: Enable the emulate-based VMX support
19be29499c52fc2861dc4b4a6492af21b6c917dd	pkvm: x86: Add enable/disable_virtualization_cpu PV interfaces	pkvm: x86: Add enable/disable_virtualization_cpu PV interfaces
0edad539f0e7bd732983ae554f41b156a44a6150	kvm: pkvm: Use PV interfaces for enable/disable_virtualization_cpu	kvm: pkvm: Use PV interfaces for enable/disable_virtualization_cpu
9a037095dd6f6781a3a044780db11560606a0157	kvm: pkvm: Set emergency_disable_virtualization_cpu callback as noop	kvm: pkvm: Set emergency_disable_virtualization_cpu callback as noop
6259693500a9cdcacd7f689d261a0d11fd5885bc	pkvm: x86: Include string lib from linux kernel	pkvm: x86: Include string lib from linux kernel
c8017bb4fbbad1ce369e84d39972997786c40499	kvm: pkvm: Use PV interface to check processor compatibility	kvm: pkvm: Use PV interface to check processor compatibility
eaf9c5d88252b9f965b6c9029579bc0e5388fc15	pkvm: x86: Add struct pkvm_vm and pkvm_vm_ref	pkvm: x86: Add struct pkvm_vm and pkvm_vm_ref
5b80f769fe8a0eee3c844417b70470a9ad3c3fe4	pkvm: vmx: Add struct pkvm_vm_vmx	pkvm: vmx: Add struct pkvm_vm_vmx
8d6d2f4b9c28e53c6bb6b54e08b0aa55b00a80ac	pkvm: x86: Move the macros for numbers of vms to a common header	pkvm: x86: Move the macros for numbers of vms to a common header
3c05f5494c5964837d349c5751ef3a7ec22eb0bf	pkvm: x86: Add vm_init/destroy PV interfaces	pkvm: x86: Add vm_init/destroy PV interfaces
1ca48813d8049d19853b88c9621f040af9cf18d7	pkvm: vmx: Implement vm_init/vm_destroy	pkvm: vmx: Implement vm_init/vm_destroy
45262d1c2aae4d8685643af0c8ec9920ff2e179f	pkvm: x86: Add pkvm_memcache to facilitate page reclaim	pkvm: x86: Add pkvm_memcache to facilitate page reclaim
01ba3c2c17c4d43b2b7b189403b982767cbaa7db	pkvm: x86: Fix another nested variadic funcall call in debug.c	pkvm: x86: Fix another nested variadic funcall call in debug.c
67a9cf20cc3e74cd5ecf8e82c34daf5c27e9dbb0	pkvm: x86: Add struct pkvm_vcpu	pkvm: x86: Add struct pkvm_vcpu
7ad43b1ebedbd5e16104018b1f3d78e4be57dc0f	pkvm: vmx: Add struct pkvm_vcpu_vmx	pkvm: vmx: Add struct pkvm_vcpu_vmx
095262d77f6d8947810595b50eca590f7b2b4df1	kvm: vmx: Change pml_pg from page pointer to virtual address	kvm: vmx: Change pml_pg from page pointer to virtual address
17992ff7a67bfece7b86d56a033caf43e39cd9e3	pkvm: vmx: Implement vcpu_create/free	pkvm: vmx: Implement vcpu_create/free
f8479f8ca1c289e77426f6bb5d81ccaedf76621e	REVERTME: pkvm: vmx: Unprotect the VMX memory pages	REVERTME: pkvm: vmx: Unprotect the VMX memory pages
35b6a6011993b3575153db66ee870715cba711d0	REVERTME: pkvm: vmx: Initialize vpid and uret msr for the host	REVERTME: pkvm: vmx: Initialize vpid and uret msr for the host
f7c8b55d99e0882c2fe501333f0b34d9e92d16e6	pkvm: x86: Add vcpu_create PV interface	pkvm: x86: Add vcpu_create PV interface
a09cd30ea398d00535fdf5f278b8e5892683885f	pkvm: x86: Add support to get/put_pkvm_vm/vcpu	pkvm: x86: Add support to get/put_pkvm_vm/vcpu
944372cc4aade6f62306df9172d00b28c56d2c01	kvm: pkvm: Refactor pkvm_shadow_vcpu_handle and shadow_vm_handle	kvm: pkvm: Refactor pkvm_shadow_vcpu_handle and shadow_vm_handle
69633d1426e3063c1c1320cb06ae7e09a006abf1	kvm: pkvm: Use PV interface for vm/vcpu life cycle management	kvm: pkvm: Use PV interface for vm/vcpu life cycle management
29166f62892e4a32c76b0eae81692910cdcbef1e	pkvm: vmx: Pass vm/vcpu handle for PKVM_HC_SET_MMIO_VE	pkvm: vmx: Pass vm/vcpu handle for PKVM_HC_SET_MMIO_VE
54c3014e9234c75f1a601545d6260f8e59460306	kvm: vmx: Remove pkvm shadow vm/vcpu life management from vt_x86_ops	kvm: vmx: Remove pkvm shadow vm/vcpu life management from vt_x86_ops
e4a4b2c0d667228c3ac07703977001a50709e5fa	pkvm: vmx: Remove unused code related to shadow vm/vcpu	pkvm: vmx: Remove unused code related to shadow vm/vcpu
a4518206b189e16c1207b271a3c36cb597eeeef3	pkvm: vmx: Remove get/put_shadow_vm	pkvm: vmx: Remove get/put_shadow_vm
b22428b86169aa425d85d0d81c524dc5da1f1903	pkvm: vmx: Disable shadow VMCS until host does vmptrld	pkvm: vmx: Disable shadow VMCS until host does vmptrld
7df0b9a88f89958b23a521be816e80e5895eafe0	pkvm: vmx: Reuse the vmcs page in vcpu_vmx struct as vmcs02	pkvm: vmx: Reuse the vmcs page in vcpu_vmx struct as vmcs02
968eda5215a4a687ee5b79f9560d7179770db26a	pkvm: vmx: Require EPT and UNRESTRICTED_GUEST features	pkvm: vmx: Require EPT and UNRESTRICTED_GUEST features
b67a5dd7f475407cc2f16c4a01f3f95071315544	kvm: vmx: Add vmptrst support	kvm: vmx: Add vmptrst support
a6fb3b207c73ad7a4d0751072f3598c20b647705	pkvm: vmx: Use vmcs_store to get the active VMCS physical address	pkvm: vmx: Use vmcs_store to get the active VMCS physical address
0e6194504cee0f7a123c5453ecd1e4ea095bf7ec	pkvm: x86: Introduce pkvm_x86_ops	pkvm: x86: Introduce pkvm_x86_ops
8dd0750e658c173cc24c4bfa94e45f27f3a93a01	pkvm: x86: Add common function to handle PV interfaces for vcpu	pkvm: x86: Add common function to handle PV interfaces for vcpu
6e3924a8e773c7d2d44051d7d1aa4ebfcdfed1e3	REVERTME: pkvm: vmx: Sync vmcs to satisfy the mixed VMX usage	REVERTME: pkvm: vmx: Sync vmcs to satisfy the mixed VMX usage
21079ca3bf5d867c3be9573792445f7e86b41059	pkvm: vmx: Implement vcpu_load	pkvm: vmx: Implement vcpu_load
91f22897632d8a32269418a55e87729368607bed	REVERTME: pkvm: vmx: Load nested vmcs to satisfy emulate-based method	REVERTME: pkvm: vmx: Load nested vmcs to satisfy emulate-based method
63fa64edf5bbb5680414f400075c0df09c2fec51	pkvm: vmx: Implement vcpu_put	pkvm: vmx: Implement vcpu_put
75ad448cf1a81ea05cd40e763e04e5f53f5265b2	REVERTME: pkvm: vmx: Release nested vmcs to satisfy emulate-based method	REVERTME: pkvm: vmx: Release nested vmcs to satisfy emulate-based method
cff23c5293fdca6d6e541db1c33fb2ad6f652ca4	pkvm: x86: Add vcpu_load PV interface	pkvm: x86: Add vcpu_load PV interface
685a7366dd74c440afcd635686072c883148cb1a	pkvm: x86: Add vcpu_put PV interface	pkvm: x86: Add vcpu_put PV interface
ce3d4a9a9862b60df6c54707d465fe620bdac944	kvm: pkvm: Use PV interfaces to load and put a vcpu	kvm: pkvm: Use PV interfaces to load and put a vcpu
c9a978707c3339e647203c2dc8a69362442adb0b	pkvm: vmx: Add dump_vmcs	pkvm: vmx: Add dump_vmcs
61f632592b6039ebdc1bdd5e01d12e3c95bae45f	pkvm: vmx: Implement cache_reg	pkvm: vmx: Implement cache_reg
70f4d72d6f739aadc01378cb8ce692f34fcdaa37	pkvm: vmx: Implement vcpu_run	pkvm: vmx: Implement vcpu_run
097b1afd65c59a3cba90b3b943a9280f6eafd376	pkvm: vmx: Make pkvm relying on virtual NMI	pkvm: vmx: Make pkvm relying on virtual NMI
44bb1b0702eb60074450586702c86dea7bff9e44	pkvm: vmx: Implement vcpu_pre_run	pkvm: vmx: Implement vcpu_pre_run
2de5097b6553d7ef5fa1a1199b738f05a1588d44	REVERTME: pkvm: vmx: Overwrite the host RIP	REVERTME: pkvm: vmx: Overwrite the host RIP
3e45f3ee03a94b9bec9af027b8e8baff5e92d8df	pkvm: x86: Add vcpu_run PV interface	pkvm: x86: Add vcpu_run PV interface
75e1d8a82e976cd4aaf4f14a749311824f020a0f	REVERTME: kvm: vmx: Always do EFER atomic switching if pkvm is enabled	REVERTME: kvm: vmx: Always do EFER atomic switching if pkvm is enabled
beb2a446a7d1773dfa352d71bdb2bfc1bb879b85	kvm: Redefine kvm print macro if runs in the pkvm hypervisor	kvm: Redefine kvm print macro if runs in the pkvm hypervisor
1b5a61885ecaa3c4b9102d4397cf4bf202562da8	pkvm: vmx: Implement handle_exit	pkvm: vmx: Implement handle_exit
33762546a021f26c185bfa1b6d739167b879be45	pkvm: x86: Handle guest vmexits in the pkvm vcpu run loop	pkvm: x86: Handle guest vmexits in the pkvm vcpu run loop
1300af1bc6f3cb15ebefb93699f348239a7f6e12	pkvm: vmx: Handle ept violation vmexit	pkvm: vmx: Handle ept violation vmexit
b5f33c4e880f2b9be50bc234ebf34d8d184f9894	pkvm: vmx: Handle init vmexit	pkvm: vmx: Handle init vmexit
2557c3d444f67a6877bb23556bda72da61359dad	pkvm: vmx: Implement skip_emulated_instruction	pkvm: vmx: Implement skip_emulated_instruction
31c4c60e72272e494c88a0975c63ce57d8f1c866	pkvm: x86: Add kvm_skip_emulated_instruction	pkvm: x86: Add kvm_skip_emulated_instruction
1ada4732869cdaa43df177fa14de3b19d8c61707	pkvm: vmx: Implement vmx_get_cpl	pkvm: vmx: Implement vmx_get_cpl
942b0629cbeefd2341af8d79dd961e614be6edaa	pkvm: vmx: Handle pVM hypercalls	pkvm: vmx: Handle pVM hypercalls
46179821af39e430dd1b23da347b0f9cd47092b7	pkvm: x86: Handle KVM_CPUID_SIGNATURE CPUID for pVM	pkvm: x86: Handle KVM_CPUID_SIGNATURE CPUID for pVM
7c38b9cfb860fcbbc305cef3413a3a7cc39a06d6	pkvm: vmx: Implement flush_tlb_all	pkvm: vmx: Implement flush_tlb_all
4838f9cce7accd5b07fe5bf43656d0729df02d11	pkvm: x86: Handle TLB_FLUSH request	pkvm: x86: Handle TLB_FLUSH request
ef0bc3c12d1c9fcf32aebc7c315d313e5052af26	pkvm: vmx: Implement flush_tlb_current	pkvm: vmx: Implement flush_tlb_current
eff16743a29573ed1e2bbc9109f70d2805f22862	pkvm: x86: Handle TLB_FLUSH_CURRENT request	pkvm: x86: Handle TLB_FLUSH_CURRENT request
e66cce57a7dfb9d8ff65746da496c2515f32bb6d	REVERTME: pkvm: vmx: Sync vcpu_vmx structure between private and shared	REVERTME: pkvm: vmx: Sync vcpu_vmx structure between private and shared
ceffa8a2f21b3a698bec5977de54c597843635b5	kvm: pkvm: Use PV interface to run vcpu	kvm: pkvm: Use PV interface to run vcpu
8eed84800146320ed7fc7d42a325b9f9f68295d7	pkvm: x86: Enhance pkvm_kick_vcpu to support guest vcpu	pkvm: x86: Enhance pkvm_kick_vcpu to support guest vcpu
6c6a4afb48f1d7b3b857e3c644c77f42fab7d380	pkvm: vmx: Enhance nested_flush_shadow_ept to support guest vcpu	pkvm: vmx: Enhance nested_flush_shadow_ept to support guest vcpu
da8997f0469b5cb23a8546a31ea72470f8d4f90e	pkvm: vmx: Use KVM_REQ_TLB_FLUSH_CURRENT to flush shadow EPT	pkvm: vmx: Use KVM_REQ_TLB_FLUSH_CURRENT to flush shadow EPT
22ba1e135f125a3333f8fe7b5add818b7d43201e	kvm: pkvm: Remove the host state settings from the pkvm_vcpu_load	kvm: pkvm: Remove the host state settings from the pkvm_vcpu_load
fb2b3c9718bc86a0e8f7f7d4fd1bb4015b5a3c67	pkvm: x86: Extend pkvm_run_vcpu to return fastpath_t to the host KVM	pkvm: x86: Extend pkvm_run_vcpu to return fastpath_t to the host KVM
09cb90750c94ddb696f5ac95cfff3f4cb8bfa4fd	pkvm: vmx: Handle force_immediate_exit	pkvm: vmx: Handle force_immediate_exit
9074b7b5a3f44ab42b0dfc3ec5d2a14f51dd3dd5	kvm: vmx: Use kvm_vcpu pointer as input for prepare_switch_to_host	kvm: vmx: Use kvm_vcpu pointer as input for prepare_switch_to_host
009c887a7484733362e07144a3ebf2d0b5d58841	pkvm: vmx: Implement prepare_switch_to_guest/host	pkvm: vmx: Implement prepare_switch_to_guest/host
3c944e134da27289aa5dd07e84a55eeb6197f69e	kvm: pkvm: Handle the uret MSRs switching	kvm: pkvm: Handle the uret MSRs switching
fabec38f4a6893e4819b77c529c235d4cb65d963	kvm: vmx: Make prepare_switch_to_host as static	kvm: vmx: Make prepare_switch_to_host as static
80faedda8ff81b41171ed5a1e817110822d3db1e	pkvm: x86: Online the uret msrs	pkvm: x86: Online the uret msrs
619dd9a7e4045de6116e7a64754a51354982c12b	pkvm: vmx: Implement set/get_msr	pkvm: vmx: Implement set/get_msr
493b25e264a61889e0ddee4940bda4273d3bfe50	pkvm: vmx: Implement complete_emulated_msr	pkvm: vmx: Implement complete_emulated_msr
aa7e336f76f125fcc12c1b0d02935cbc148a6796	pkvm: x86: Implement kvm_emulate_rdmsr/wrmsr	pkvm: x86: Implement kvm_emulate_rdmsr/wrmsr
a930c44f178c4b688277fc1392dc7b0cff0e8a93	pkvm: vmx: Switching the uret MSRs in the pkvm hypervisor	pkvm: vmx: Switching the uret MSRs in the pkvm hypervisor
9069905ea7f12e89b5acbdf7dacfa8655fdfd2cf	pkvm: vmx: Implement vcpu_after_set_cpuid	pkvm: vmx: Implement vcpu_after_set_cpuid
d84de5175246f6ef4c0e4788fa64774d22e0dc4d	pkvm: x86: Add vcpu_after_set_cpuid PV interface	pkvm: x86: Add vcpu_after_set_cpuid PV interface
c12a2a8996d7cb20f88c39e16d4e678b0401bfa1	pkvm: x86: Disable the VMX feature for the guest	pkvm: x86: Disable the VMX feature for the guest
8b60ca60298506a635625c69df20e75138a3b8ca	pkvm: x86: Disable all PV features	pkvm: x86: Disable all PV features
94c268bf8cb1f4a1e6956d7c96be7963f0f7f6be	kvm: pkvm: Use PV interface to do vcpu_after_set_cpuid	kvm: pkvm: Use PV interface to do vcpu_after_set_cpuid
0308609b57a63a0930ae70769d03e9b9d937f219	pkvm: vmx: Implement interrupt injection callbacks	pkvm: vmx: Implement interrupt injection callbacks
b4e005a423c71e1241c98fd9875bef4539b65a36	pkvm: vmx: Implement nmi injection callbacks	pkvm: vmx: Implement nmi injection callbacks
3a9a80c695813b6f8b505e3875c3454064795cac	pkvm: vmx: Implement inject_exception callback	pkvm: vmx: Implement inject_exception callback
b6db735f619028ff002f4d1164a24b8463f89f72	pkvm: vmx: Implement set_rflags	pkvm: vmx: Implement set_rflags
cceb6f92ef208fd9c29e58e4bc80034100ae5dc5	pkvm: vmx: Implement set_dr7	pkvm: vmx: Implement set_dr7
3f4723331ea94e1c4ff067901cfc380878ce1f43	pkvm: x86: Handle the KVM_REQ_EVENT request	pkvm: x86: Handle the KVM_REQ_EVENT request
95bd4bea063d26332e5764da4ab5f0c7fc1e8f73	REVERTME: pkvm: vmx: Sync irq/nmi/exception related fields	REVERTME: pkvm: vmx: Sync irq/nmi/exception related fields
331e20d25b66f8c0f5f17c7098de1c06812a5427	pkvm: x86: Implement kvm_multiple_exception	pkvm: x86: Implement kvm_multiple_exception
9bc5b0ff59499bc768c07279ca98a56bdb72e698	pkvm: vmx: Inject #GP to guest if CPL > 0 when handles the vmcall	pkvm: vmx: Inject #GP to guest if CPL > 0 when handles the vmcall
40cc4a7bbe49f02282f105c9e40e8ee79efa4f7b	pkvm: vmx: Implement vmx_complete_interrupt	pkvm: vmx: Implement vmx_complete_interrupt
2dce4d2573a6cf24d0b86508699fcfd25c0aa7cf	pkvm: x86: Handle MSR emulation error in the pkvm hypervisor	pkvm: x86: Handle MSR emulation error in the pkvm hypervisor
64c8cfa6187d33c23ba4336c7277aad3d1185afd	pkvm: vmx: Implement vcpu_reset	pkvm: vmx: Implement vcpu_reset
77b218ec4ba46b9772e1c95539457ee44011367d	pkvm: vmx: Implement set_efer	pkvm: vmx: Implement set_efer
682250c24440dc7c3a9757a9154ec3e710a946d1	pkvm: vmx: Implement set_cr4	pkvm: vmx: Implement set_cr4
39bad4ee91e12f5ea206ccaee4d535c5b7c09d27	pkvm: vmx: Implement set_cr0	pkvm: vmx: Implement set_cr0
17fe96f48f486e2e0d0e5ddca778534380bb9d5f	pkvm: x86: Add kvm_vcpu_reset	pkvm: x86: Add kvm_vcpu_reset
610ea8af09ed08453546275e316ec622953db242	kvm: pkvm: Use PV interface to do vcpu reset	kvm: pkvm: Use PV interface to do vcpu reset
f7ebfcdec453eb85effffbb498ccab75814d3d17	Revert "REVERTME: pkvm: vmx: Overwrite the host RIP"	Revert "REVERTME: pkvm: vmx: Overwrite the host RIP"
564c31c5f371b1e921c90c88b5dab68f7d65e705	kvm: pkvm: Add union pkvm_pv_param	kvm: pkvm: Add union pkvm_pv_param
fa732a828760c81bcb4a43743c89900242d78eaa	pkvm: vmx: Implement get/set_segment	pkvm: vmx: Implement get/set_segment
827363ac6ab49e4a1b3ff9df87646d35e01d28c2	kvm: pkvm: Use PV interfaces to access guest segment	kvm: pkvm: Use PV interfaces to access guest segment
e8005a0e9812bee2b49f9b6be8aa7e20041bf3eb	kvm: pkvm: Cache the segment	kvm: pkvm: Cache the segment
a12bf3715e666041726dd193a68b98c1d3a53f12	kvm: pkvm: Use PV interface to set_cr0/cr4	kvm: pkvm: Use PV interface to set_cr0/cr4
9bebf6d154e9978d68c79a5342517acb98745bd4	kvm: pkvm: Implement is_valid_cr0/cr4 callbacks	kvm: pkvm: Implement is_valid_cr0/cr4 callbacks
6fb80d8b84854be655060dbeb105044f2dbcc9e6	pkvm: x86: Add set/get_msr PV interface	pkvm: x86: Add set/get_msr PV interface
089a4ac1e1233026d83d541b6dda13b23b8cad87	kvm: pkvm: Use PV interface to access guest MSRs	kvm: pkvm: Use PV interface to access guest MSRs
b8a579d908eb1ec5818ada6881080b285598491a	kvm: pkvm: Disable update_exception_bitmap	kvm: pkvm: Disable update_exception_bitmap
e6c35da539fee2b02c718fa6393fb8adeaef2f08	kvm: pkvm: Use PV interface to set_efer	kvm: pkvm: Use PV interface to set_efer
85ffe2cd762ff5ed7d2bbb5f37dee57984c21a3c	kvm: pkvm: Use PV interface to access idt/gdt	kvm: pkvm: Use PV interface to access idt/gdt
d8e3876afb97b466ba9fd37b407435ed62c9dfec	kvm: pkvm: Use PV interface to set dr7	kvm: pkvm: Use PV interface to set dr7
7f7959d638ce41c1f6de13fa8742b1a746d3b259	kvm: pkvm: Use PV interface to access rflags	kvm: pkvm: Use PV interface to access rflags
d71239a93c0b642f27b8fa7042376ac9461b9935	kvm: pkvm: Use PV interface to flush tlb	kvm: pkvm: Use PV interface to flush tlb
dd689ddbbce71e9b5eb613e7aa915bc9a697da4c	kvm: pkvm: Use PV interface to get/set interrupt shadow	kvm: pkvm: Use PV interface to get/set interrupt shadow
88e27311175105ac70c73cff348fed723c05887b	pkvm: x86: Add request HOST_HANDLE_EXIT	pkvm: x86: Add request HOST_HANDLE_EXIT
d0ca719abd91b1c934760e3366b2908040a7c6ca	pkvm: x86: Sync npVM vcpu GPRs between the pkvm hypervisor and the host	pkvm: x86: Sync npVM vcpu GPRs between the pkvm hypervisor and the host
efd06c995dc1d1e3047e0b8f69a97bc7db0084e9	pkvm: x86: Set pVM vcpu initial state for boot	pkvm: x86: Set pVM vcpu initial state for boot
9c142dd0b6531db6e087a521768ce522ef813ae9	pkvm: x86: Add sync_vcpu_state_post_switch	pkvm: x86: Add sync_vcpu_state_post_switch
e82bade5149244c93ef06f2b43199ac3b205f2a0	pkvm: x86: Add sync_vcpu_state_pre_switch	pkvm: x86: Add sync_vcpu_state_pre_switch
6f8fac236b60cff0b6fdc51ae4bfd23ee775323d	pkvm: vmx: Share pVM vcpu state to the host for emulating MSR	pkvm: vmx: Share pVM vcpu state to the host for emulating MSR
1a84e4fd62e85cae8f9bdda95819b642fc756df8	kvm: pkvm: Clear the injected nmi_injected/interrupt queue	kvm: pkvm: Clear the injected nmi_injected/interrupt queue
105cd4deb66e5f6e4ef0a5329145eee3f5b21973	Revert "REVERTME: pkvm: vmx: Sync irq/nmi/exception related fields"	Revert "REVERTME: pkvm: vmx: Sync irq/nmi/exception related fields"
7686106c81ec09d79d40ee39b387699e6c641846	kvm: pkvm: Use PV interface to check if irq injection is allowed or not	kvm: pkvm: Use PV interface to check if irq injection is allowed or not
54a4950c1d965c0fd0a04fd28ca3a660e21866b0	kvm: pkvm: Use PV interface to check if nmi injection is allowed or not	kvm: pkvm: Use PV interface to check if nmi injection is allowed or not
9aa70c9501bb5e7380f8cbef7c897813fbc83bec	kvm: pkvm: Use PV interface to inject interrupt	kvm: pkvm: Use PV interface to inject interrupt
557bd5574e18d503c7eb29578f8c2de41f4ba20c	kvm: pkvm: Use PV interface to inject nmi	kvm: pkvm: Use PV interface to inject nmi
269bc04d4ce4374bd10cf4206231825d6394ff99	kvm: pkvm: Use PV interface to inject exception for the npVM	kvm: pkvm: Use PV interface to inject exception for the npVM
34d84ea91ae814459f2f943e0ea79906804f787f	kvm: pkvm: Use PV interface to cancel injection	kvm: pkvm: Use PV interface to cancel injection
12a73fb9a1fe1a944d768699323006656d4e4d81	kvm: pkvm: Add PV interface to get/set nmi mask	kvm: pkvm: Add PV interface to get/set nmi mask
305ea2ab1ee5c8b96d1ea04f5fe88c2d7867ac0a	kvm: pkvm: Use PV interface to enable nmi window	kvm: pkvm: Use PV interface to enable nmi window
35c3ddd28512fa9847631b53d6bc3f87c0564385	kvm: pkvm: Use PV interface to enable irq window	kvm: pkvm: Use PV interface to enable irq window
47e00faf48666b91df76efdb6ef347c5826a8b3d	kvm: pkvm: Use PV interface to update_cr8_intercept	kvm: pkvm: Use PV interface to update_cr8_intercept
3520b93c78e3c0dbd582acf0eb98d67e7a06870a	kvm: pkvm: Enable x2apic mode by default for the pVM	kvm: pkvm: Enable x2apic mode by default for the pVM
f50d337a6a71739de5f08db6cfddc56504bdb809	pkvm: vmx: Implement set_virtual_apic_mode	pkvm: vmx: Implement set_virtual_apic_mode
78bc8ce2cadc05418e78436cb957657f5d2a5dbf	kvm: pkvm: Use PV interface to set virtual APIC mode	kvm: pkvm: Use PV interface to set virtual APIC mode
7f424f9513fcd9ea32cd87d3286d3f2da60b2e91	kvm: pkvm: Not to use virtual apic access	kvm: pkvm: Not to use virtual apic access
e9a54df7a962d075147e56b44837b0322b92380d	kvm: pkvm: Use PV interface to refresh apicv exec ctrl	kvm: pkvm: Use PV interface to refresh apicv exec ctrl
bf84c1ee3955bffdd976167da2acc7782fc2cb86	kvm: pkvm: Use PV interface to load eoi exit bitmap	kvm: pkvm: Use PV interface to load eoi exit bitmap
e18a37b2c621fa5bd89ddc5a5785b12c67f71b1e	kvm: x86: Add kvm_vcpu as input for hwapic_isr_update ops	kvm: x86: Add kvm_vcpu as input for hwapic_isr_update ops
679fb05cb2897e16f340692330938e31520624f8	kvm: pkvm: Use PV interface to update hwapic isr/irr	kvm: pkvm: Use PV interface to update hwapic isr/irr
ea97337b5060149b930638673df848bc31541569	kvm: pkvm: Add its own sync_pir_to_irr	kvm: pkvm: Add its own sync_pir_to_irr

dd3fd802ffe870aca1c70f801be92e86a724e8f3	pkvm: vmx: Return CS and SS AR bytes to the host	pkvm: vmx: Return CS and SS AR bytes to the host
0748d9bbcc38d2aa381929aa6e3403a3b4e5cb78	kvm: pkvm: Decode cpl and cs_db_l from the cached seg	kvm: pkvm: Decode cpl and cs_db_l from the cached seg
97003da9a1a631f59e6d228fdf100de3c8e95424	kvm: pkvm: Get the exit information from vcpu_vmx structure	kvm: pkvm: Get the exit information from vcpu_vmx structure
193ea0166e3aad597b33056ee9b1a422e5893520	kvm: pkvm: Use PV interface to write tsc	kvm: pkvm: Use PV interface to write tsc
93f10c6940bd24304c550f6767a2cdf53edda8c4	kvm: pkvm: Post set guest cr3 via the PV interface	kvm: pkvm: Post set guest cr3 via the PV interface
f0febc1826e78309f2c6e009ec42b3a7bda34be3	pkvm: vmx: Implement load_mmu_pgd	pkvm: vmx: Implement load_mmu_pgd
85f019a6df82cd61f1444cbfe25b89a31366c2f5	pkvm: x86: Add setup_virtual_mmu ops for pkvm_x86_ops	pkvm: x86: Add setup_virtual_mmu ops for pkvm_x86_ops
5e3e24f1def1a9d7aa1ddf7dede2ebdf2a0a9634	pkvm: x86: Add load_mmu_pgd PV interface	pkvm: x86: Add load_mmu_pgd PV interface
a22790b302cabb91c103371245e658b55891355f	kvm: pkvm: Use PV interface to load mmu	kvm: pkvm: Use PV interface to load mmu
5a95efe96cfa9d3748360b335e5c2be3f6a92f42	kvm: pkvm: Use PV interface to setup mce	kvm: pkvm: Use PV interface to setup mce
bfb501842fe043539d8faba5fb0dd003025caf0d	kvm: pkvm: Disallow the host to change MSR exit bitmap	kvm: pkvm: Disallow the host to change MSR exit bitmap
695418bad8b57fef62fc88e3b566f7d8cbd6d3f8	pkvm: vmx: Remove the task_switch vmexit handler	pkvm: vmx: Remove the task_switch vmexit handler
ab497a99f4a80af60b83ecd1a1635c987ed2e28a	pkvm: vmx: Remove the desc vmexit handler	pkvm: vmx: Remove the desc vmexit handler
3fcf2353f0b498781d48de5de83824f19d981941	pkvm: vmx: Remove PML vmexit handler	pkvm: vmx: Remove PML vmexit handler
eee089d6a35e70b1238ce5e075f6fdd8970530d0	pkvm: x86: Handle IA32_CR_PAT MSR vmexit	pkvm: x86: Handle IA32_CR_PAT MSR vmexit
63677e108cb73f56ccf70a6b5d8b8ed633197efd	pkvm: x86: Handle IA32_MISC_ENABLE MSR vmexit	pkvm: x86: Handle IA32_MISC_ENABLE MSR vmexit
8cc685292897a8a4e728306bdbf84f6c962a3bca	pkvm: vmx: Handle xsetbv vmexit	pkvm: vmx: Handle xsetbv vmexit
b96234e949dada3b8eb9c5695229554086d74508	pkvm: x86: Handle cpuid vmexit	pkvm: x86: Handle cpuid vmexit
e7e7229b1fa8f5329df9fe32f83bf2b3a960e41c	pkvm: vmx: Handle DR access vmexit	pkvm: vmx: Handle DR access vmexit
8e4d03729d16d9dd44adf11c81bbfd4aeebadec4	pkvm: x86: Handle invd vmexit	pkvm: x86: Handle invd vmexit
a41b63142f29584a8c8c8046290c0f8e85865c44	pkvm: x86: Handle rdpmc vmexit	pkvm: x86: Handle rdpmc vmexit
c12676ee70911fc6093c0816b2c2ec6f2c2cdf75	pkvm: x86: Handle rdrand/rdseed vmexits	pkvm: x86: Handle rdrand/rdseed vmexits
a41019b60d4d09f97c4a5394277f64e0adaa1eaf	pkvm: x86: Handle mwait/monitor vmexit	pkvm: x86: Handle mwait/monitor vmexit
4ee4f36d6868309b6c51867248c80f5f5766d472	pkvm: vmx: Handle monitor trap vmexit	pkvm: vmx: Handle monitor trap vmexit
d5c2fe9306908cdaf8dcdd64bb19c6589317cddc	pkvm: vmx: Handle vmx instruction vmexits	pkvm: vmx: Handle vmx instruction vmexits
51358b9b71b97ab04a30597a8be39ad19bde753b	pkvm: vmx: Handle invlpg/invpcid vmexit	pkvm: vmx: Handle invlpg/invpcid vmexit
ee8f82bd475cef4379218cfeefb2b0e9d630ea48	pkvm: vmx: Handle encls vmexit	pkvm: vmx: Handle encls vmexit
ce4bd314a8a6b9fd8d6b42396df6c1296490a347	pkvm: vmx: Require TPR shadow	pkvm: vmx: Require TPR shadow
87a6576998af659708e879c33de352109021bfda	pkvm: vmx: Handle CR vmexit	pkvm: vmx: Handle CR vmexit
8023b719ddcd0468c6e9e57f8b9e5c8df4bdc6d1	pkvm: vmx: Deliver pVM's cr0/cr4/efer to the host KVM to init/reset MMU	pkvm: vmx: Deliver pVM's cr0/cr4/efer to the host KVM to init/reset MMU
47b6917433c9759e128544bcc20de9d61406f39e	pkvm: x86: Handle EFER MSR vmexit	pkvm: x86: Handle EFER MSR vmexit
5473ce952a678adbc14aa919be36657c18a8ba45	Revert "REVERTME: kvm: vmx: Always do EFER atomic switching if pkvm is enabled"	Revert "REVERTME: kvm: vmx: Always do EFER atomic switching if pkvm is enabled"
f64d30c9fb8981f76e2811ad9b761ffb0c785672	REVERTME: pkvm: vmx: Handle the MSR_IA32_XFD MSR vmexit	REVERTME: pkvm: vmx: Handle the MSR_IA32_XFD MSR vmexit
bef156edd9b766c1262d85563506af3a35a6af7d	pkvm: vmx: Handle the NM exception	pkvm: vmx: Handle the NM exception
ed23f3b2976a0738a381461d515b94c2406d23b1	pkvm: vmx: Only intercept MC exception for the protected VM by default	pkvm: vmx: Only intercept MC exception for the protected VM by default
51ce041779aaf5cdf1302c7e663a443682c07deb	pkvm: vmx: Add the original linux vmexit handler code for comparison	pkvm: vmx: Add the original linux vmexit handler code for comparison
dc928a0ee84e088c2593fc017cbd7e2cc7acce83	kvm: pkvm: Set guest state protection flags for pVM	kvm: pkvm: Set guest state protection flags for pVM
12f791b13e928b8811c059c1eb8633c8781fb36a	pkvm: x86: Enforce pVM vcpu state protection to the PV interfaces	pkvm: x86: Enforce pVM vcpu state protection to the PV interfaces
3c96e9dc98710b1ad8cfa0486f1cea982a1a89b5	pkvm: vmx: Enforce protection to pVM's CS and SS AR byte	pkvm: vmx: Enforce protection to pVM's CS and SS AR byte
2b9b075cee99d7a961086905a41f0e53feeeb682	kvm: pkvm: Add handle_exit in the host KVM	kvm: pkvm: Add handle_exit in the host KVM
48efbdc266aea7a8b27d77775170b1138a6e87ef	kvm: pkvm: Implement check_emulate_instruction	kvm: pkvm: Implement check_emulate_instruction
a0c6916c64a1223ce7f9acc6cc65ab8d0d30f4c8	pkvm: vmx: Handle interrupt window vmexit	pkvm: vmx: Handle interrupt window vmexit
53a4c42c23d43ef2f47bab54396cc9ee5b0e75f8	pkvm: vmx: Handle nmi window vmexit	pkvm: vmx: Handle nmi window vmexit
6193f8ebe1d2e36d53b983380182fe63fddbb502	pkvm: vmx: Handle hlt vmexit	pkvm: vmx: Handle hlt vmexit
0a65045a8aace42fbfabba945b2cec2307ab0c0d	pkvm: vmx: Handle the pause vmexit	pkvm: vmx: Handle the pause vmexit
4540b2486cec342cd7adadb7f019f22aaae4b5b2	pkvm: vmx: Handle EPT_MISCONFIG vmexit	pkvm: vmx: Handle EPT_MISCONFIG vmexit
6d4e88b59847db50a7f730da361eee5e49420169	pkvm: vmx: Handle EPT_VIOLATION vmexit	pkvm: vmx: Handle EPT_VIOLATION vmexit
d3fc5a865cd5cc1ca1c466376802bc713c031198	pkvm: vmx: Not to skip the instruction for EPT_MISCONFIG/VIOLATION	pkvm: vmx: Not to skip the instruction for EPT_MISCONFIG/VIOLATION
cb5b0fd8f79501dffa746ed68c319f868b389657	pkvm: vmx: Handle IO vmexit	pkvm: vmx: Handle IO vmexit
23c1c2ff743e1755c5596d9be571c2b62df0f179	pkvm: vmx: Share pVM vcpu state to the host for handling vmcall	pkvm: vmx: Share pVM vcpu state to the host for handling vmcall
690a5457370dcdfc726dfdc773c674994d5454dc	pkvm: vmx: Handle wbinvd vmexit	pkvm: vmx: Handle wbinvd vmexit
7ef4dd017d6e0369ad69bddd3a73aab0df3b8c39	pkvm: vmx: Handle bus lock vmexit	pkvm: vmx: Handle bus lock vmexit
40b25b0c9f85033d5089822511607ebe07c58a08	pkvm: vmx: Handle notify vmexit	pkvm: vmx: Handle notify vmexit
a916fa4f6f1bc94cf118677133569d3a9c1ac284	pkvm: x86: Switch the debug registers	pkvm: x86: Switch the debug registers
eff6ec35c29698f88168df8652a5ae286b166079	kvm: pkvm: Noop the sync_dirty_debug_regs callback	kvm: pkvm: Noop the sync_dirty_debug_regs callback
f104e430d0a5081ff7f9cf3b048ff291396af26f	kvm: pkvm: Use PV interface to cache register	kvm: pkvm: Use PV interface to cache register
95e54d86cb1fa4f13b55e1dbdd564533061695db	kvm: pkvm: Add skip instruction function	kvm: pkvm: Add skip instruction function
6bc974ef110ec244caca5df21526680268d2d277	Revert "REVERTME: pkvm: vmx: Sync vcpu_vmx structure between private and shared"	Revert "REVERTME: pkvm: vmx: Sync vcpu_vmx structure between private and shared"
1417b067b6d6536067e0acf5665444e271b5f078	Revert "REVERTME: pkvm: vmx: Release nested vmcs to satisfy emulate-based method"	Revert "REVERTME: pkvm: vmx: Release nested vmcs to satisfy emulate-based method"
21e176f45ec008bf7d08c883c402bcb7c79589b1	Revert "REVERTME: pkvm: vmx: Load nested vmcs to satisfy emulate-based method"	Revert "REVERTME: pkvm: vmx: Load nested vmcs to satisfy emulate-based method"
16bb96c5ddddbea920c90bf24d1c975a3cc33103	Revert "REVERTME: pkvm: vmx: Sync vmcs to satisfy the mixed VMX usage"	Revert "REVERTME: pkvm: vmx: Sync vmcs to satisfy the mixed VMX usage"
4486f36403d082c5deca2c7dc0fc2aa63b74ac9c	Revert "REVERTME: pkvm: vmx: Initialize vpid and uret msr for the host"	Revert "REVERTME: pkvm: vmx: Initialize vpid and uret msr for the host"
eb7e1539d331f9e290937f377ce9cb55bea8225b	Revert "REVERTME: pkvm: vmx: Unprotect the VMX memory pages"	Revert "REVERTME: pkvm: vmx: Unprotect the VMX memory pages"
6f6aacd41bd48226652e6decd72eee0bb8232772	Revert "REVERTME: pkvm: vmx: Enable the emulate-based VMX support"	Revert "REVERTME: pkvm: vmx: Enable the emulate-based VMX support"
482b34d6dc0fe97b4c7c60efb0888e51e250870f	pkvm: vmx: Remove VMX emulation	pkvm: vmx: Remove VMX emulation
774aef18df8daa06e54cca7a4ee8d00ebb766249	pkvm: vmx: Use POSTED_INTR_DESC_ADDR with the shared pi_desc	pkvm: vmx: Use POSTED_INTR_DESC_ADDR with the shared pi_desc
811255f19e95ade37157f85d3c8f535f779cea15	kvm: pkvm: Initialize pi_desc	kvm: pkvm: Initialize pi_desc
6cf22e8b68ef0b8e4fb1fb1fa7207d595268d54f	kvm: pkvm: Not to update RVI if the vector is -1	kvm: pkvm: Not to update RVI if the vector is -1
db6f52a4eb4c1ebe4bc8d9c6163a2ea80ba0e956	kvm: pkvm: Add update_cpuid_runtime PV interface	kvm: pkvm: Add update_cpuid_runtime PV interface
dd33e76c92bb41dd9a13603ded6286799c877628	kvm: pkvm: Add uret MSRs to the pkvm hypervisor emulated MSR list	kvm: pkvm: Add uret MSRs to the pkvm hypervisor emulated MSR list
518ae9711764e19d0eab670426207c7c41722611	pkvm: vmx: Support guest single-setup debug exception	pkvm: vmx: Support guest single-setup debug exception
1759c69442c190794f0e2492286b2284216f9f28	pkvm: x86: Disable PV features only for the pVM	pkvm: x86: Disable PV features only for the pVM
b8ae63213caa2ded1f378cdf91b671727182e932	kvm: pkvm: Remove some nested_ops	kvm: pkvm: Remove some nested_ops
b91a42252256c7ec78b48fa3e1cdbab918efb5d1	kvm: pkvm: Check exception.pending	kvm: pkvm: Check exception.pending
219ed44e0bb0d8d3b533f6cb8fc0978b409c8b8e	kvm: pkvm: Use PV interface to update exception bitmap	kvm: pkvm: Use PV interface to update exception bitmap
42b18d5e97afa3d87c4207da119b191724eb1777	pkvm: x86: Update switch_db_regs when set DR7	pkvm: x86: Update switch_db_regs when set DR7
e01c0564e498fafca81324466f199f46d058e298	pkvm: x86: Enhance the debug registers switching	pkvm: x86: Enhance the debug registers switching
453d5d498c9fe571d339e914b6a3be6fbd642405	pkvm: x86: Sync debug registers for the npVM	pkvm: x86: Sync debug registers for the npVM
12981cd960c20f3cdcf0545a6714d9d55dcb3713	pkvm: x86: Support guest singlestep debug	pkvm: x86: Support guest singlestep debug
53f8c898bc3435db4c0f1f3fe52f6eb27b7cb657	pkvm: vmx: Refactor the npvm state sharing for vmexits handling	pkvm: vmx: Refactor the npvm state sharing for vmexits handling
357e1982d19280f87078ed4e4a1877ad465a480a	kvm: pkvm: Support handling DB/BP exceptions for guest debug	kvm: pkvm: Support handling DB/BP exceptions for guest debug
7f8673c940f8493418de9b744e9132cf7617ced9	kvm: pkvm: Handle DR_ACCESS vmexit to support guest debug	kvm: pkvm: Handle DR_ACCESS vmexit to support guest debug
be86c564582c7236a66337c214d1885528e71c12	pkvm: vmx: Fix including kvm-asm-offsets.h when compiling with O=	pkvm: vmx: Fix including kvm-asm-offsets.h when compiling with O=
8561161d4900a38e404e0372fc41952ba7d277c8	pkvm: vmx: Fix including pkvm_constants.h when compiling with O=	pkvm: vmx: Fix including pkvm_constants.h when compiling with O=
b03f7f7d8261fd8947ac901871b9e78f2ddb6b84	pkvm: x86: Fix clang -Wsection warning for __cpu_possible_mask	pkvm: x86: Fix clang -Wsection warning for __cpu_possible_mask
435bf561f8b6391444aacfaa5b598f8148e0a846	pkvm: vmx: Fix clang -Wc23-extensions warning in vmx_vcpu_enter_exit()	pkvm: vmx: Fix clang -Wc23-extensions warning in vmx_vcpu_enter_exit()
b52d36f697f328755ecee2687b742ec456151d7d	pkvm: x86: Fix possible uninitialized return value in pkvm_vcpu_create()	pkvm: x86: Fix possible uninitialized return value in pkvm_vcpu_create()
02e4f9dc269d618358c4e9fec801e3bc63e5aae2	KVM: x86: Get vcpu->arch.apic_base directly and drop kvm_get_apic_base()	KVM: x86: Get vcpu->arch.apic_base directly and drop kvm_get_apic_base()
d69ce85a8b83baf8292ce95b6886a65e79044d4e	KVM: x86: Inline kvm_get_apic_mode() in lapic.h	KVM: x86: Inline kvm_get_apic_mode() in lapic.h
cee9eda539c3544b2844fbb1dbc006351e631526	pkvm: x86: Fix "error: redefinition of 'kvm_get_apic_mode'"	pkvm: x86: Fix "error: redefinition of 'kvm_get_apic_mode'"
5322a48795cee84cb8c687bf37e872d355c375b4	Use single symbol as parameter for pkvm_sym()	Use single symbol as parameter for pkvm_sym()
e453d117f42307a75275b0738fe929030e84ccd0	Separate function name as a parameter in PKVM_DECLARE	Separate function name as a parameter in PKVM_DECLARE
00292f1b88d6ac193498c015b103cd96aadb2cb2	Use macro for debug symbol extension	Use macro for debug symbol extension
c4aec5c5648c00331d9310f85bd3be36ed964261	Change prefix to suffix in patching global symbols	Change prefix to suffix in patching global symbols
b61a271778b1f7aac6651b2174c35f659c1abb10	Enable CFI in pkvm compilation if it is enabled in config	Enable CFI in pkvm compilation if it is enabled in config
03ac3be8a990f8f7384ea118ee4d20409a57163a	pkvm: x86: Fix incorrect target file path	pkvm: x86: Fix incorrect target file path
20f79e36ebf3909c0aef3509ce5f2b8f739f5fea	KVM: x86: Load DR6 with guest value only before entering .vcpu_run() loop	KVM: x86: Load DR6 with guest value only before entering .vcpu_run() loop
824c93b7de9bdb415efaefa56aa9360531057e2f	pkvm: x86: Load DR6 in x86 core	pkvm: x86: Load DR6 in x86 core
5d8559fac031d1c303c1f5400a0c093f53dec190	pkvm: x86: Delete two useless parameters of objcopy	pkvm: x86: Delete two useless parameters of objcopy
bf67bfa64b321a5fc784d62e959bc03260578b75	pkvm: x86: update objcopy for adding suffix	pkvm: x86: update objcopy for adding suffix
1fe8ea12d446ff2f65b67eba354156de9e459aef	pkvm: x86: Fix occasional compiling failures for the kvm-asm-offsets.h	pkvm: x86: Fix occasional compiling failures for the kvm-asm-offsets.h
78a77e3de22c4206bf413b27631dc1c4b9a62dc7	pkvm: vmx: Remove extra printk in CPU deprivilege	pkvm: vmx: Remove extra printk in CPU deprivilege
b57460e18b688dc2f50ec65e4278dbb1a6227e93	pkvm: x86: Avoid using lockdep in pKVM hyp code	pkvm: x86: Avoid using lockdep in pKVM hyp code
7e6a060daa9ef2df8b86d6d38e5162921a5a43ce	pkvm: x86: Add #undef CONFIG_TRACE_IRQFLAGS in def.h	pkvm: x86: Add #undef CONFIG_TRACE_IRQFLAGS in def.h
4c79dbf5c4db111a807738e0806b41c546a4b193	pkvm: x86: Add #undef CONFIG_DEBUG_IRQFLAGS in def.h	pkvm: x86: Add #undef CONFIG_DEBUG_IRQFLAGS in def.h
c8d5df9e7e1d4cda3e3157f685c7d6267be9c864	pkvm: vmx: Remove irq disable/enable in this_cpu_do_finalise_hc()	pkvm: vmx: Remove irq disable/enable in this_cpu_do_finalise_hc()
298d6c71f28b0efea0a021533f442130c050483c	pkvm: x86: Fix missing spinlock init for pinned_page_lock	pkvm: x86: Fix missing spinlock init for pinned_page_lock
67962f3cb01bab6c15b3889ef99b3aad00f62bb3	Merge pull request #29 from maluka-dmytro/pvVMCS-POC-v6.12-runtime-lockdep-fixes	Merge pull request #29 from maluka-dmytro/pvVMCS-POC-v6.12-runtime-lockdep-fixes
f955ef6af03b41a541fac6e01173a28b83c0596f	kvm: pkvm: Fix using sleeping function when skip instruction	kvm: pkvm: Fix using sleeping function when skip instruction
5beea09850cba5e0932cf7580d5887bce8571e75	KVM: x86: Snapshot the host's DEBUGCTL in common x86	KVM: x86: Snapshot the host's DEBUGCTL in common x86
1bcfd34b0dfb362a134793c8073e007bffda96f5	KVM: x86: Snapshot the host's DEBUGCTL after disabling IRQs	KVM: x86: Snapshot the host's DEBUGCTL after disabling IRQs
d427fd3421c661d297000584df50ff64c5d1eb61	pkvm: x86 align to recent kvm changes which modified vcpu_vmx struct	pkvm: x86 align to recent kvm changes which modified vcpu_vmx struct
eef31bc7aa8480d314eb417957efbeaa1c3e4390	pkvm: x86: Prevent the vcpu loading race	pkvm: x86: Prevent the vcpu loading race
1bd34210bbda75dbf287c04c4c1dae7cd11b2dd6	pkvm: x86: Prevent access pkvm_vcpu on a wrong CPU	pkvm: x86: Prevent access pkvm_vcpu on a wrong CPU
9fdcd212f2a83716ee9fd0a60e7a56357f833343	pkvm: vmx: Disable ipiv via setting enable_ipiv to false	pkvm: vmx: Disable ipiv via setting enable_ipiv to false
5439aead18b08c3442c00dfbd7f5bcc63704ccc8	kvm: pkvm: Change the percpu pv_param as a structure	kvm: pkvm: Change the percpu pv_param as a structure
dd89018f4ddcf60987df57232b20734258795524	ANDROID: pkvm: x86: Add support for custom unmap_leaf callback	ANDROID: pkvm: x86: Add support for custom unmap_leaf callback
24d383ee49197d9823e56448d9ffd275c637adb9	ANDROID: pkvm: x86: Prevent host from donating an IOMMU mapped page	ANDROID: pkvm: x86: Prevent host from donating an IOMMU mapped page
38c824d787065c87723c339267759075a7fb4db0	ANDROID: pkvm: x86: Avoid using refcounting for reserved memory	ANDROID: pkvm: x86: Avoid using refcounting for reserved memory
e5f870208fe64272e51f91d3356b6e10d440af01	ANDROID: pkvm: x86: Allow scalable mode without nested translation	ANDROID: pkvm: x86: Allow scalable mode without nested translation
89e7ac74fb19f067936407292d70d40d8b764dab	ANDROID: pkvm: x86: Use INVALID_GPA instead of PVMFW_INVALID_LOAD_ADDR	ANDROID: pkvm: x86: Use INVALID_GPA instead of PVMFW_INVALID_LOAD_ADDR
8005e9c16340990930b8378d335e312ee2ca20b4	ANDROID: pkvm: x86: Refactor pvmfw handling as a part of new PV code	ANDROID: pkvm: x86: Refactor pvmfw handling as a part of new PV code
ea6f88175efc5203afcbfce0b151e5c8833b5d7d	ANDROID: pkvm: vmx: Remove vcpu_pre_run	ANDROID: pkvm: vmx: Remove vcpu_pre_run
1deca56c852adf3668663d3f07426725d979dd38	ANDROID: pkvm: x86: Properly return error in pvmfw address set ioctl	ANDROID: pkvm: x86: Properly return error in pvmfw address set ioctl
06806d405b35b71c9f9f00b26e4fadcb3f6ee819	ANDROID: pkvm: x86: Enforce secure pvmfw bootstrap	ANDROID: pkvm: x86: Enforce secure pvmfw bootstrap
112f820a9a4ee919dd90884b69723d2c964a4a9c	ANDROID: pkvm: x86: Clean up pre-configuring vCPU by host	ANDROID: pkvm: x86: Clean up pre-configuring vCPU by host
97e57c61c49facabdc8fdac69709765411ade786	ANDROID: pkvm: x86: Disable FRED for the pkvm hypervisor	ANDROID: pkvm: x86: Disable FRED for the pkvm hypervisor
feca41c436cb2f4b834517a531f4bb943681f4f4	ANDROID: pkvm: x86: Refactor the exception/NMI handlers	ANDROID: pkvm: x86: Refactor the exception/NMI handlers
edbba593827db5c2adf647f2ee858fc1d4f60f2c	ANDROID: kvm: pkvm: Fix the unbalanced guest_state_enter/exit_irqoff()	ANDROID: kvm: pkvm: Fix the unbalanced guest_state_enter/exit_irqoff()
158ff8dd7dd1156a6dfac501c6a11ba194ea1123	ANDROID: pkvm: vmx: Fix non-working erase of pvmfw when pkvm is disabled	ANDROID: pkvm: vmx: Fix non-working erase of pvmfw when pkvm is disabled
5464aabc7555527ea4a9880acf30981880c93810	ANDROID: pkvm: vmx: Protect most of hyp memory in debug mode	ANDROID: pkvm: vmx: Protect most of hyp memory in debug mode
5c5b159ca0b6d29d257738b3cb44c3766020f37d	ANDROID: pkvm: vmx: Disable IRQs on first CPU in this_cpu_do_finalise_hc()	ANDROID: pkvm: vmx: Disable IRQs on first CPU in this_cpu_do_finalise_hc()
f4a8046dafab4d75d8aa8a78919c1adbddea4b3b	ANDROID: pkvm: x86: Don't use e820__get_entry_type() for pvmfw check	ANDROID: pkvm: x86: Don't use e820__get_entry_type() for pvmfw check
7738b4b0d27ddbc305ab5ed57c98c3390de8bccd	ANDROID: pkvm: x86: Update comment about CC_FLAGS_CFI	ANDROID: pkvm: x86: Update comment about CC_FLAGS_CFI
7eadd68ad145dbd454b7dd088f3b2b53dd1f3e59	x86/its: Enumerate Indirect Target Selection (ITS) bug	x86/its: Enumerate Indirect Target Selection (ITS) bug
21c567c8baa1a2a68caf29787ef365ec6c31a3e1	ANDROID: pkvm: x86: Move pkvm image variables to pkvm_image.h	ANDROID: pkvm: x86: Move pkvm image variables to pkvm_image.h
702b0e0a33fdf65e0a8b50b8d6fe0f8706c6c49d	ANDROID: pkvm: x86: Define the pkvm percpu data in the kernel's percpu section	ANDROID: pkvm: x86: Define the pkvm percpu data in the kernel's percpu section
b0696b57934ec3a90a3b395ba2fb367115ff3c47	ANDROID: pkvm: x86: use gs as percpu base for both debug and non-debug build	ANDROID: pkvm: x86: use gs as percpu base for both debug and non-debug build
fb094cab0a471c7041f70fee05b22e695ee7a0d8	ANDROID: pkvm: x86: Rename the smp.c to cpu.c	ANDROID: pkvm: x86: Rename the smp.c to cpu.c
f7833a91a92a85f6f3de1c17c3761e41cb2fb5bb	ANDROID: pkvm: x86: reuse retpoline.S in kernel for pkvm	ANDROID: pkvm: x86: reuse retpoline.S in kernel for pkvm
361cd4d1946e864769db3c6c4e53e5b383908579	ANDROID: pkvm: x86: Undefine certain kernel configs for the ASM code	ANDROID: pkvm: x86: Undefine certain kernel configs for the ASM code
d0e71d0bd3559333d8f4739cb5fdc408140a295a	ANDROID: pkvm: x86: Retpoline rework in pkvm	ANDROID: pkvm: x86: Retpoline rework in pkvm
cc6784e9aa659ae592485a3b55b362163a298f2b	pkvm: x86: Align with android16-6.12-desktop branch	pkvm: x86: Align with android16-6.12-desktop branch
b5e53dbfd25b4c1824bf98a9ab7b2f0b7d2b6773	ANDROID: pkvm: x86: Support devices requiring ATS	ANDROID: pkvm: x86: Support devices requiring ATS
cd68edf319371039e0d0d0c834bef6179f134ed6	ANDROID: DEBUG: pkvm: x86: Add the guest vmexit tracing	ANDROID: DEBUG: pkvm: x86: Add the guest vmexit tracing
2cbe37f35abad713b85a75f783680c7c3e32babb	ANDROID: DEBUG: pkvm: x86: Enhance the max vmexit reason number for trace	ANDROID: DEBUG: pkvm: x86: Enhance the max vmexit reason number for trace
a24ad2474ba879b9874efcc2bbf6b86f82c82672	ANDROID: DEBUG: pkvm: x86: Use per vcpu vmexit_perf to trace guest vmexit	ANDROID: DEBUG: pkvm: x86: Use per vcpu vmexit_perf to trace guest vmexit
023ba17ff77d7a01f4a74322239c4b47278239c6	ANDROID: DEBUG: pkvm: x86: Dump vmexit trace to stdout	ANDROID: DEBUG: pkvm: x86: Dump vmexit trace to stdout
a7d4d0473d9749ee2a241d93f60ca0fce66d7497	ANDROID: DEBUG: pkvm: x86: Add per-vm pkvm_vmexit_trace	ANDROID: DEBUG: pkvm: x86: Add per-vm pkvm_vmexit_trace
eb30b3f3a694d6c2bf05435ef3d1e395309a6675	ANDROID: kvm: pkvm: Check and set kvm governed features	ANDROID: kvm: pkvm: Check and set kvm governed features
511c908a9aad41a69ca70889bc62f8dcd505fcb6	ANDROID: pkvm: x86: Passthrough the PCI configuration space	ANDROID: pkvm: x86: Passthrough the PCI configuration space
916fc57bf1646d41898309764dba2f518978d145	ANDROID: pkvm: x86: Fix IOMMU MMIO register access issue from host VM	ANDROID: pkvm: x86: Fix IOMMU MMIO register access issue from host VM
af4e6665db4eafef1b6ee5ec43782ac77c2b4453	ANDROID: pkvm: vmx: Enable the CPU vulnerabilities mitigation code for vmenter.S	ANDROID: pkvm: vmx: Enable the CPU vulnerabilities mitigation code for vmenter.S
3a5fdeb2dceb4aa33b45c51775c31d654c2f2565	ANDROID: pkvm: vmx: Push vcpu_vmx in the pkvm RSP rather than kvm_vcpu	ANDROID: pkvm: vmx: Push vcpu_vmx in the pkvm RSP rather than kvm_vcpu
8bf19b9fac13ab01e67f2993e3ac1e7ad2c01b62	ANDROID: pkvm: vmx: Switch the spec ctrl MSR when switching to/from the host	ANDROID: pkvm: vmx: Switch the spec ctrl MSR when switching to/from the host
973ae84c392df77b89425695dd7e601242d8a45c	ANDROID: pkvm: vmx: Add BHI/MDS mitigations for running the host VM	ANDROID: pkvm: vmx: Add BHI/MDS mitigations for running the host VM
a49f022fc8f66c930fc77be2f7c9bcda671f97ba	ANDROID: pkvm: x86: Add l1d flush pages	ANDROID: pkvm: x86: Add l1d flush pages
1f30efe72c824ce8e982dbcefeb8cc55e27d0521	ANDROID: pkvm: vmx: Hide l1d contents of the pVM from the host	ANDROID: pkvm: vmx: Hide l1d contents of the pVM from the host
2beab3b7f4c7b40fcd6b6e5fcd260b6c497dfd65	ANDROID: pkvm: vmx: Flush l1d for the guest in the pkvm hypervisor	ANDROID: pkvm: vmx: Flush l1d for the guest in the pkvm hypervisor
d8c0b3635c322bec02c7789539ed373608e51588	ANDROID: pkvm: x86: wait for qi quiesce only if events in flight	ANDROID: pkvm: x86: wait for qi quiesce only if events in flight
597c1c5c2492f4f7f13e626acc0d51332fc5e4a8	ANDROID: pkvm: x86: re-privilege cpus on deprivilege failure	ANDROID: pkvm: x86: re-privilege cpus on deprivilege failure
60f38ad6b2d3ccb9423ab18c8437cdaa49341bf0	ANDROID: pkvm: x86: delay host iommu driver initialization after deprivilage	ANDROID: pkvm: x86: delay host iommu driver initialization after deprivilage
cd36d9d7e5aaa872449e04df6debae699238f64e	ANDROID: pkvm: x86: unmap iommu mmio space during host iommu enable	ANDROID: pkvm: x86: unmap iommu mmio space during host iommu enable
c8b85cba3b5e5becb4300282e5f32e9b687465bf	ANDROID: pkvm: x86: wrapper function for pkvm enabled check in host	ANDROID: pkvm: x86: wrapper function for pkvm enabled check in host
184d6c15d6a852e37b9de61acd95c62633ff681d	ANDROID: pkvm: vmx: Inject #GP into host on failed EPT violation	ANDROID: pkvm: vmx: Inject #GP into host on failed EPT violation
4e77db4856e9256001414f31f8cf87ef5ac48169	ANDROID: pkvm: vmx: Remove duplicate vmx_enable_irq_window() impl	ANDROID: pkvm: vmx: Remove duplicate vmx_enable_irq_window() impl
966ba32739eae8f5d79573de7e9d9d3810c351e2	ANDROID: pkvm: vmx: Improve and fix up NMI injection to host	ANDROID: pkvm: vmx: Improve and fix up NMI injection to host
fa7d6fd42e691d0e6b51a487059b6e4506fa3e87	ANDROID: pkvm: vmx: Enforce PMU isolation	ANDROID: pkvm: vmx: Enforce PMU isolation
8cfcc58f28fd217b96c3285fa8ca702b8f617d01	KVM: x86: drop x86.h include from cpuid.h	KVM: x86: drop x86.h include from cpuid.h
4e8d122481b704a6a0e562162cf386582ca6842c	KVM: x86: Route non-canonical checks in emulator through emulate_ops	KVM: x86: Route non-canonical checks in emulator through emulate_ops
5a4437c5893e7ae47a05ae381c8527ddfaea06ec	KVM: x86: Add X86EMUL_F_MSR and X86EMUL_F_DT_LOAD to aid canonical checks	KVM: x86: Add X86EMUL_F_MSR and X86EMUL_F_DT_LOAD to aid canonical checks
619b1badf09a5f282d52ffd161bc8868c879fe7e	KVM: x86: model canonical checks more precisely	KVM: x86: model canonical checks more precisely
363cb1703d6940e1d3b053b4be96c1f48ea8a91c	KVM: x86/hyper-v: Skip non-canonical addresses during PV TLB flush	KVM: x86/hyper-v: Skip non-canonical addresses during PV TLB flush
35cfcafa07917246ef0b51456d2758b02518cfc1	KVM: x86: Free vCPUs before freeing VM state	KVM: x86: Free vCPUs before freeing VM state
a1d908b6911d8ff80f77547cc6c274c582436768	KVM: x86: Convert vcpu_run()'s immediate exit param into a generic bitmap	KVM: x86: Convert vcpu_run()'s immediate exit param into a generic bitmap
9d783d0e9977c2599894bf19bab4335594a064ef	KVM: x86: Drop kvm_x86_ops.set_dr6() in favor of a new KVM_RUN flag	KVM: x86: Drop kvm_x86_ops.set_dr6() in favor of a new KVM_RUN flag
ee90be1c0ed08aa7b7e7f9f03294bc3af11062ca	KVM: VMX: Allow guest to set DEBUGCTL.RTM_DEBUG if RTM is supported	KVM: VMX: Allow guest to set DEBUGCTL.RTM_DEBUG if RTM is supported
9ea333da87b080a92fcbd112a3aeb85465fecd63	KVM: VMX: Extract checking of guest's DEBUGCTL into helper	KVM: VMX: Extract checking of guest's DEBUGCTL into helper
7c82870fb333a06cc907eb294506f9cb46558dc4	KVM: nVMX: Check vmcs12->guest_ia32_debugctl on nested VM-Enter	KVM: nVMX: Check vmcs12->guest_ia32_debugctl on nested VM-Enter
2a091619cf4f542a523bf154f301e7f37a63a33a	KVM: VMX: Wrap all accesses to IA32_DEBUGCTL with getter/setter APIs	KVM: VMX: Wrap all accesses to IA32_DEBUGCTL with getter/setter APIs
384315cd963d3b4f579f3032bd81f7ca7c68b6ab	KVM: VMX: Preserve host's DEBUGCTLMSR_FREEZE_IN_SMM while running the guest	KVM: VMX: Preserve host's DEBUGCTLMSR_FREEZE_IN_SMM while running the guest
f3aa4d57798eefba84189cf1d7f4b984c9b2ceb9	KVM: x86: use array_index_nospec with indices that come from guest	KVM: x86: use array_index_nospec with indices that come from guest
c23dc07f9f7b69d85d26c531c7170e0709902c06	ANDROID: pkvm: x86: Update vcpu_run() to use run_flags	ANDROID: pkvm: x86: Update vcpu_run() to use run_flags
eda6751fb360b42ececb4fd1b0bb2563ef76f729	ANDROID: pkvm: x86: Drop .set_dr6()	ANDROID: pkvm: x86: Drop .set_dr6()
0f6d0f570bcf9cf49133eb84ef2c8260eebee27b	ANDROID: pkvm: x86: Update is_noncanonical_address() usage	ANDROID: pkvm: x86: Update is_noncanonical_address() usage
d67a78bff2c286e3bcf932089a3164df188d87a4	ANDROID: pkvm: x86: Update vmx_get_supported_debugctl()	ANDROID: pkvm: x86: Update vmx_get_supported_debugctl()
6d4383bf73ee91b93fc042791249c4919098f186	ANDROID: pkvm: x86: Unload vCPU when freeing it	ANDROID: pkvm: x86: Unload vCPU when freeing it
7e7851105901d9e02b420934ff71954ada44d2d4	ANDROID: pkvm: x86: Fix spectre-v1 in free_pkvm_vm_handle	ANDROID: pkvm: x86: Fix spectre-v1 in free_pkvm_vm_handle
69a5a23403a8987247ad0dde641b58ec30827166	ANDROID: pkvm: x86: Fix spectre-v1 in get_pkvm_vm	ANDROID: pkvm: x86: Fix spectre-v1 in get_pkvm_vm
08aba426bea9904ad80323d56e0f087fad27ea66	ANDROID: pkvm: x86: Fix spectre-v1 in get_pkvm_vcpu_from_vm	ANDROID: pkvm: x86: Fix spectre-v1 in get_pkvm_vcpu_from_vm
19cea85aff96a3ce230fd0c16c2e39ee6d737328	ANDROID: pkvm: vmx: Add IBPB when switching between host VM and guest VM	ANDROID: pkvm: vmx: Add IBPB when switching between host VM and guest VM
bf9b43dc46651f0dd381991dbdd5734b3b350b85	ANDROID: pkvm: x86: Prevent pkvm to run if CPU has unsupported bugs	ANDROID: pkvm: x86: Prevent pkvm to run if CPU has unsupported bugs
04840e3b3b8ae97605f47922e7807f5877f9e9e8	ANDROID: pkvm: x86: Add kernel command line to control cpu_bug_relax	ANDROID: pkvm: x86: Add kernel command line to control cpu_bug_relax
8674f4cabe435943cb8843a77f6c2a96325ed7a0	ANDROID: pkvm: x86: set PKVM_INTEL depend on !BLK_DEV_FD	ANDROID: pkvm: x86: set PKVM_INTEL depend on !BLK_DEV_FD
d16058d528250f06725dcd33bc5bff67ab24a988	ANDROID: pkvm: x86: Remove SUPPRESS_VE macro	ANDROID: pkvm: x86: Remove SUPPRESS_VE macro
47270b3a418cf2d4b11ff7bc3a8f7ad1ca63aca0	ANDROID: pkvm: x86: Allow flush_tlb callback set to NULL	ANDROID: pkvm: x86: Allow flush_tlb callback set to NULL
1b1e90ad94ded60c7c31fb1c60dc2debb6c530d9	ANDROID: pkvm: x86: Use const pointers for page table ops	ANDROID: pkvm: x86: Use const pointers for page table ops
e3eff03f7d033904771788048e319108e60e1313	ANDROID: pkvm: vmx: Implement EPT support for PV MMU	ANDROID: pkvm: vmx: Implement EPT support for PV MMU
96088c0d9a58d22a351746ef9f55d926fd06c5ff	ANDROID: pkvm: x86: Add PV MMU page table	ANDROID: pkvm: x86: Add PV MMU page table
dc6b4fb323fa666e8cb7e902f6d5d63454b827f1	ANDROID: pkvm: vmx: Add vmx_get_mt_mask()	ANDROID: pkvm: vmx: Add vmx_get_mt_mask()
c83482e68bef50f2dbb2a7daef6fe91fae99a736	ANDROID: pkvm: x86: Add inline to pkvm_hypercall() definition	ANDROID: pkvm: x86: Add inline to pkvm_hypercall() definition
2c79e62d1f5a1208deca735790547a053f84df1d	ANDROID: pkvm: x86: Increase max number of pKVM hypercall arguments	ANDROID: pkvm: x86: Increase max number of pKVM hypercall arguments
e7f29980e53fc2b0d943c9981b37c05147cb9a73	ANDROID: pkvm: x86: REVERTME: Further increase max num of hypercall args	ANDROID: pkvm: x86: REVERTME: Further increase max num of hypercall args
26907c4bacc690a35a75b5fa4026716622c2d0ce	ANDROID: pkvm: x86: Disable enable_mmio_caching	ANDROID: pkvm: x86: Disable enable_mmio_caching
d15b346363b577cd6c86d973804b3997d3526d7e	ANDROID: pkvm: x86: Implement PV MMU map hypercall	ANDROID: pkvm: x86: Implement PV MMU map hypercall
ceeb35a32fd45b2c7f81aeebacf843da1be7fafd	ANDROID: pkvm: x86: Implement removing PV MMU mappings on VM teardown	ANDROID: pkvm: x86: Implement removing PV MMU mappings on VM teardown
c80d67f3bd74256d19f6d100d9049ec46fd671c7	ANDROID: pkvm: x86: Implement TLB shootdown for PV MMU	ANDROID: pkvm: x86: Implement TLB shootdown for PV MMU
e940e01a0f195d4ef467dc09ac991cfc82328fc7	ANDROID: pkvm: x86: Implement host PV MMU page fault handler	ANDROID: pkvm: x86: Implement host PV MMU page fault handler
74fe6f51b10b25b48acabc30431c60afa71a1ca6	ANDROID: pkvm: x86: Check pvmfw load address for overflow	ANDROID: pkvm: x86: Check pvmfw load address for overflow
aee08df08cc4352434c8d26b85ac9bd7e7f1e4a3	ANDROID: pkvm: x86: Implement copying pvmfw to pVM memory with PV MMU	ANDROID: pkvm: x86: Implement copying pvmfw to pVM memory with PV MMU
8aa5ac0a5475caf1ed155f5f8bd1b29450408c39	ANDROID: pkvm: x86: Implement PV MMU unmap hypercall	ANDROID: pkvm: x86: Implement PV MMU unmap hypercall
d769f46c0b75f77ca8f05beae3e796ad1343c281	ANDROID: pkvm: x86: Add pkvm_pgtable_test_clear_young()	ANDROID: pkvm: x86: Add pkvm_pgtable_test_clear_young()
77ca5942c8e058f752f4bea51664a6b048fdbc17	ANDROID: pkvm: vmx: Remove redundant enable_ept check	ANDROID: pkvm: vmx: Remove redundant enable_ept check
4db42946318f40dd2fb0a88f752108956b52b71d	ANDROID: pkvm: vmx: Return -EOPNOTSUPP for missing capabilities	ANDROID: pkvm: vmx: Return -EOPNOTSUPP for missing capabilities
b2d80f703e74102527dd077c6e4d17f8c1d97ff2	ANDROID: pkvm: vmx: Implement page age support for EPT	ANDROID: pkvm: vmx: Implement page age support for EPT
36d18d1fc05c4a1a4f1b2952974f649b999702c4	ANDROID: pkvm: x86: Implement PV MMU age hypercall	ANDROID: pkvm: x86: Implement PV MMU age hypercall
306e685d911ed3fc5cc6a2ae41fa15f7d672577e	ANDROID: pkvm: x86: Plumb MMU notifiers in PV MMU	ANDROID: pkvm: x86: Plumb MMU notifiers in PV MMU
3203d9f6e79a26926cb507e08ae54665067d708a	ANDROID: pkvm: x86: Skip kvm_arch_flush_shadow_all() with pKVM	ANDROID: pkvm: x86: Skip kvm_arch_flush_shadow_all() with pKVM
04c0a00ef352cde10e0d894e8d6a7e4395d2244b	FROMLIST: KVM: x86/mmu: Skip MMIO SPTE invalidation if enable_mmio_caching=0	FROMLIST: KVM: x86/mmu: Skip MMIO SPTE invalidation if enable_mmio_caching=0
3e80b71c8f7c7c7bb184a181924a334838163618	ANDROID: pkvm: x86: Don't use zap_all quirk with pKVM	ANDROID: pkvm: x86: Don't use zap_all quirk with pKVM
b12c23c16fa9d19eeb8eadacde4696807e096ceb	ANDROID: pkvm: x86: Disable NX huge page workaround	ANDROID: pkvm: x86: Disable NX huge page workaround
95eac7499553717a130a056764b9c9f0ba0e5d19	ANDROID: pkvm: x86: Disable dirty logging	ANDROID: pkvm: x86: Disable dirty logging
4ee253a293521735a18257cccfe0de6138f56680	ANDROID: pkvm: x86: Disable page tracking	ANDROID: pkvm: x86: Disable page tracking
8ed2449303432753729cc8d441ddc042005cf161	ANDROID: pkvm: x86: Switch to using PV MMU instead of emulated EPT	ANDROID: pkvm: x86: Switch to using PV MMU instead of emulated EPT
be444596b8b3f7cc0d05f4c10ad12e807b39d25a	ANDROID: pkvm: x86: Handle guest EPT violation fully in host	ANDROID: pkvm: x86: Handle guest EPT violation fully in host
6185c8cf06309ec16deefca4c997be26582fa9b3	ANDROID: pkvm: x86: Don't allocate vEPT in host	ANDROID: pkvm: x86: Don't allocate vEPT in host
62e8bab9bd4af1a43e9597eccf08f3444d7fe71b	ANDROID: pkvm: x86: Clean up pVM page pinning	ANDROID: pkvm: x86: Clean up pVM page pinning
2671c36638070d9ca83dc7c0b77c7fd696a4d41c	ANDROID: pkvm: x86: Warn if root_to_sp() is called	ANDROID: pkvm: x86: Warn if root_to_sp() is called
9194408260ef9a4eee76ec5088f4a65c05a8940e	ANDROID: pkvm: x86: Remove FIXMEs about HOST_INIT_MMU/HOST_RESET_MMU	ANDROID: pkvm: x86: Remove FIXMEs about HOST_INIT_MMU/HOST_RESET_MMU
9f89dc6dd321dac3bf454bb71b62e13f86049fe7	ANDROID: pkvm: x86: Don't share CR0/CR4/EFER with host for pVM	ANDROID: pkvm: x86: Don't share CR0/CR4/EFER with host for pVM
78ae5bf9cd0f925a973c96177de31f680496e177	ANDROID: pkvm: x86: Remove FIXME about kvm_mmu_after_set_cpuid()	ANDROID: pkvm: x86: Remove FIXME about kvm_mmu_after_set_cpuid()
25e25735642e7ce9c06fd7c682360feb46e6b267	ANDROID: pkvm: x86: Remove FIXME about MMU reset in kvm_vcpu_reset()	ANDROID: pkvm: x86: Remove FIXME about MMU reset in kvm_vcpu_reset()
040403d2df3710dc732f00d9e5642ad3da525661	ANDROID: pkvm: x86: Remove FIXME about TLB flushing interfaces	ANDROID: pkvm: x86: Remove FIXME about TLB flushing interfaces
c454ec953dc05001c656d14df24cbb15eac61eb6	ANDROID: pkvm: x86: Don't allow host to flush TLB of a running pVM	ANDROID: pkvm: x86: Don't allow host to flush TLB of a running pVM
617bacf5fc0a0250b814bbbbb918e97c74a03ec1	ANDROID: pkvm: x86: remove not used pkvm_handle_guest_ept_violation and its dependencies	ANDROID: pkvm: x86: remove not used pkvm_handle_guest_ept_violation and its dependencies
da1cd3c0b11a405104198495c5d1f3eaf5bf5edc	ANDROID: pkvm: x86: remove deprecated shadow page table initialization	ANDROID: pkvm: x86: remove deprecated shadow page table initialization
7bbd1cd448c089069616259241c71f1af3d01759	ANDROID: pkvm: x86: remove virtual ept setup	ANDROID: pkvm: x86: remove virtual ept setup
75925af0161eb8a821ac6ef36df7c51f571bb8dc	ANDROID: pkvm: x86: remove dead code related to shadow ept invalidation	ANDROID: pkvm: x86: remove dead code related to shadow ept invalidation
56ef87f4dc854c9691afdb2fc627f1f313bd4c2e	Revert "ANDROID: KVM: stats: Add VM stat for remote tlb flush with range"	Revert "ANDROID: KVM: stats: Add VM stat for remote tlb flush with range"
eb59316d29cf71ceea2666fa467bb47402137b2c	Revert "ANDROID: pkvm: x86: flush remote TLBs with range in mmu_notifier callbacks"	Revert "ANDROID: pkvm: x86: flush remote TLBs with range in mmu_notifier callbacks"
040023779f3a793dc990f9bd3049cfb0b5fddd3b	Revert "ANDROID: pkvm: x86: Fix compiling issue when CONFIG_HYPERV=n"	Revert "ANDROID: pkvm: x86: Fix compiling issue when CONFIG_HYPERV=n"
22c66f20b91c13575271893f55563dc7c6f3cc48	ANDROID: pkvm: x86: remove deprecated pvmfw load logic	ANDROID: pkvm: x86: remove deprecated pvmfw load logic
b37589a83e2dd6d962c95bb94ba31a9b92326716	ANDROID: pkvm: x86: remove not initialized SEPT leftovers	ANDROID: pkvm: x86: remove not initialized SEPT leftovers
dcb99214ae15607c99f8b43d2964e86101b2d6ad	Revert "ANDROID: pkvm: x86: Initialize every SEPT entry with "suppress #VE" bit set"	Revert "ANDROID: pkvm: x86: Initialize every SEPT entry with "suppress #VE" bit set"
222b8d389b7d6c816a2afb4c739e6454f08d7d7c	Revert "ANDROID: pkvm: x86: Enable Virtualization Exception Information"	Revert "ANDROID: pkvm: x86: Enable Virtualization Exception Information"
97070ea67e500a1709425bb40b93083adad54122	ANDROID: pkvm: x86: Refactor the pkvm_vcpu attach/detach	ANDROID: pkvm: x86: Refactor the pkvm_vcpu attach/detach
6a4792ef2254a3aee497ee73346d2f46aa62fd13	ANDROID: pkvm: x86: Add a vcpu reference count array in pkvm_vm	ANDROID: pkvm: x86: Add a vcpu reference count array in pkvm_vm
14b9c5d271401932ebdf99f805f57d67b5905ee8	ANDROID: pkvm: x86: Add vcpu_free PV interface	ANDROID: pkvm: x86: Add vcpu_free PV interface
fed2113b1881bf97737e1bb54a7d318c1cdb6169	ANDROID: pkvm: x86: Prevent host from freeing a loaded pkvm_vcpu	ANDROID: pkvm: x86: Prevent host from freeing a loaded pkvm_vcpu
38a95b7cc18b24863eaf5242e003cf5a31baf053	ANDROID: pkvm: x86: Check NULL pkvm_vcpu when traversing pkvm_vm->vcpus	ANDROID: pkvm: x86: Check NULL pkvm_vcpu when traversing pkvm_vm->vcpus
7516c4437f90e3da5cb48eccfc46bce7b0f44bfe	ANDROID: kvm: pkvm: Free pkvm_vcpu via the vcpu_free PV interface	ANDROID: kvm: pkvm: Free pkvm_vcpu via the vcpu_free PV interface
0cf5d64d9c0c37ab2541ae5ce4a97be04d524df5	ANDROID: pkvm: x86: Extend __pkvm_hyp_donate_host to clear memory pages	ANDROID: pkvm: x86: Extend __pkvm_hyp_donate_host to clear memory pages
a39def1de1b3467d5c5cff6470710c1bb5cbabf9	ANDROID: pkvm: x86: Free the old cpuid entries memory correctly	ANDROID: pkvm: x86: Free the old cpuid entries memory correctly
2f551231b35b1731798f3578a2debb9ede87e8df	ANDROID: pkvm: x86: Setup fpu kernel and user config for the pkvm hypervisor	ANDROID: pkvm: x86: Setup fpu kernel and user config for the pkvm hypervisor
563e41a110f95b4c6071290f74055b5559129977	ANDROID: pkvm: x86: Setup xstate offset and size for the pkvm hypervisor	ANDROID: pkvm: x86: Setup xstate offset and size for the pkvm hypervisor
d0190090df4e7ae0e1abdaca8d82bd95b4bec861	ANDROID: pkvm: x86: Use setup_pkvm_per_cpu to setup percpu for debug/non-debug	ANDROID: pkvm: x86: Use setup_pkvm_per_cpu to setup percpu for debug/non-debug
67598a1826948ef4a792ad560de28f2858ae5f00	ANDROID: pkvm: x86: Add percpu fpu	ANDROID: pkvm: x86: Add percpu fpu
7ecc59c22bd4e6704c191ccfac78da4719b13702	ANDROID: pkvm: x86: Allocate the fpstate for the guest vcpu	ANDROID: pkvm: x86: Allocate the fpstate for the guest vcpu
5494c15b0395be7ecb1eed362e35291281555f8b	ANDROID: pkvm: x86: Initialize the guest fpu	ANDROID: pkvm: x86: Initialize the guest fpu
2cdbbbe68ede62bd87c49643b7012af913fcdcdf	ANDROID: pkvm: x86: Add new PV interface to reallocate fpstate	ANDROID: pkvm: x86: Add new PV interface to reallocate fpstate
1631b37bb9e97cb61fce8e2751b8f5d31b4083b9	ANDROID: pkvm: x86: Add support to enable guest xfd features	ANDROID: pkvm: x86: Add support to enable guest xfd features
23e66a4604dd806161a63afb6ceb4a3cdb86dcaf	ANDROID: pkvm: x86: Add the fpu register swapping functions	ANDROID: pkvm: x86: Add the fpu register swapping functions
b7bc02ad8d3ffe3e5e148785e04c93c8c252ba7c	ANDROID: pkvm: x86: Swap the FPU registers for the pVM	ANDROID: pkvm: x86: Swap the FPU registers for the pVM
f27519404d5cf4e38270ad7f11f3f26d5b277b57	ANDROID: pkvm: x86: Refactor the vcpu run loop	ANDROID: pkvm: x86: Refactor the vcpu run loop
54b62008c28f4408e79b19f46bec09e0731d259e	ANDROID: pkvm: x86: Handle MSR_IA32_XFD and MSR_IA32_XFD_ERR	ANDROID: pkvm: x86: Handle MSR_IA32_XFD and MSR_IA32_XFD_ERR
0e07fde6349c20144d67a33bf6a2b2603a43701f	ANDROID: Revert "ANDROID: REVERTME: pkvm: vmx: Handle the MSR_IA32_XFD MSR vmexit"	ANDROID: Revert "ANDROID: REVERTME: pkvm: vmx: Handle the MSR_IA32_XFD MSR vmexit"
191d160fee1caa2303e4c98799e905a00b60757e	ANDROID: pkvm: x86: Save the host_pkru when load a vcpu	ANDROID: pkvm: x86: Save the host_pkru when load a vcpu
0d8ae6e08133a453f4d6fcb25583929cb26ddeb1	ANDROID: pkvm: x86: Support clearing BNDREGS/BNDCSR components in the fpstate	ANDROID: pkvm: x86: Support clearing BNDREGS/BNDCSR components in the fpstate
4784571a9ccfa579799b1e71a7f7ba0ad1d203d8	ANDROID: pkvm: x86: Add kvm_vcpu_deliver_sipi_vector()	ANDROID: pkvm: x86: Add kvm_vcpu_deliver_sipi_vector()
2251b90eb9a9965a251caf67043ffc114ee7fc0c	ANDROID: pkvm: x86: Implement secure startup of secondary vCPUs	ANDROID: pkvm: x86: Implement secure startup of secondary vCPUs
2123ae57986a86537abbcb7705c7bd665b984c5e	ANDROID: pkvm: x86: Hotfix non-working SMP in old guest kernels	ANDROID: pkvm: x86: Hotfix non-working SMP in old guest kernels
1bd6828a9aba485cd5f740075e52481548c9a5c8	ANDROID: pkvm: vmx: Don't WARN_ON if pVM executes string IO instruction	ANDROID: pkvm: vmx: Don't WARN_ON if pVM executes string IO instruction
7651aa2802de5cec4ba2f75f390edef5ca3b3bd5	ANDROID: pkvm: x86: Emulate more MSRs in the pkvm hypervisor	ANDROID: pkvm: x86: Emulate more MSRs in the pkvm hypervisor
c7b43662284b5d574382e438a32c4ad4d06a9ebe	ANDROID: pkvm: x86: Emulate MTRR MSRs in the pkvm hypervisor	ANDROID: pkvm: x86: Emulate MTRR MSRs in the pkvm hypervisor
9ba9d62af575d9d835f67f052743de4175aba14c	ANDROID: pkvm: x86: Emulate MCE MSRs in the pkvm hypervisor	ANDROID: pkvm: x86: Emulate MCE MSRs in the pkvm hypervisor
920c5691866cba320ef65e4a9f8100e7d8436a1b	ANDROID: pkvm: x86: add "memcache" argument to zalloc_page()	ANDROID: pkvm: x86: add "memcache" argument to zalloc_page()
2114d792f7bbffbdad3ed3838816fffb300f397e	ANDROID: pkvm: x86: hyp: prepare infrastructure to use memcache	ANDROID: pkvm: x86: hyp: prepare infrastructure to use memcache
7991733e9876e0b24a4b0f09c49d825c664e2a57	ANDROID: pkvm: x86: introduce kvm_protected_vcpu to group pKVM fields	ANDROID: pkvm: x86: introduce kvm_protected_vcpu to group pKVM fields
f36cee4d054def2b00f896c05793700c619c537d	ANDROID: pkvm: x86: hyp: switch to memcache version of zalloc	ANDROID: pkvm: x86: hyp: switch to memcache version of zalloc
68e34d4465f5c91f94a165994802c7679e99e143	ANDROID: pkvm: x86: introduce guest mmu memcache and use it	ANDROID: pkvm: x86: introduce guest mmu memcache and use it
310aea6150f6941447537933fe9348bab16b41c9	ANDROID: pkvm: x86: make free_pkvm_memcache global	ANDROID: pkvm: x86: make free_pkvm_memcache global
3ae379338c9cfe6b5f6a4d0e2503219f14b20c92	ANDROID: pkvm: x86: fill memcache with host backed pages	ANDROID: pkvm: x86: fill memcache with host backed pages
499806fb8481e75e8467a419a9d41a2f710d6775	ANDROID: pkvm: x86: handle memcache refill during page sharing	ANDROID: pkvm: x86: handle memcache refill during page sharing
a755cd778212775af0f5c6c8f41875d920113af1	ANDROID: pkvm: x86: stop relying on shadow pgt pool	ANDROID: pkvm: x86: stop relying on shadow pgt pool
992b048595db3191a1565a1b111f5fdb1b1e36b7	ANDROID: pkvm: x86: do not preallocate pages for shadow EPT	ANDROID: pkvm: x86: do not preallocate pages for shadow EPT
141bce2e53dd4212a91d1ae65c01714320c191e4	ANDROID: pkvm: x86: minor header cleanups	ANDROID: pkvm: x86: minor header cleanups
ad6d6b08977fdf6d7762bc3802fd176731fd72ac	ANDROID: pkvm: x86: Add host memory pages sharing/unsharing with pKVM	ANDROID: pkvm: x86: Add host memory pages sharing/unsharing with pKVM
de11dd370a0bc346b6ed3c1483a4658293b56968	ANDROID: pkvm: x86: Add pin/unpin shared memory page API	ANDROID: pkvm: x86: Add pin/unpin shared memory page API
e7977379dea642542d29ac6d8e06d4be5e2cf383	ANDROID: pkvm: x86: Pin/unpin memory pages shared-owned with guest	ANDROID: pkvm: x86: Pin/unpin memory pages shared-owned with guest
cc4bfeaa458da9ad31359fc0bbe183db035928d6	ANDROID: pkvm: x86: Expose share/unshare with hyp via PV interface	ANDROID: pkvm: x86: Expose share/unshare with hyp via PV interface
c5fa4e21dd7c5d74acc982abf8077bd9cdeb7ce0	ANDROID: pkvm: x86: Add host KVM logic to share and unshare memory	ANDROID: pkvm: x86: Add host KVM logic to share and unshare memory
871dcd400fb9d99b771d1e794a12c343b37e2b29	ANDROID: kvm: pkvm: Share and unshare kvm structure with pKVM	ANDROID: kvm: pkvm: Share and unshare kvm structure with pKVM
0fbf7af013baedbc6b9cc312861b6f17068a6726	ANDROID: kvm: pkvm: Share and unshare kvm_vcpu with pKVM	ANDROID: kvm: pkvm: Share and unshare kvm_vcpu with pKVM
0f63adabca5b9dee7a76bbbd2659685b42b56f2e	ANDROID: pkvm: x86: Add actual kvm_vm/kvm_vcpu size	ANDROID: pkvm: x86: Add actual kvm_vm/kvm_vcpu size
a1f3b702ac994d005a2d972277bfb68abf77ca9d	ANDROID: pkvm: x86: Pin and unpin kvm structure	ANDROID: pkvm: x86: Pin and unpin kvm structure
58b5e9333f51fa8a828b6dc8056d2e3c5b86304c	ANDROID: pkvm: x86: Pin and unpin shared kvm_vcpu	ANDROID: pkvm: x86: Pin and unpin shared kvm_vcpu
710468c29629481275083d6eb0b3e876e9ed1658	ANDROID: pkvm: x86: Share/pin and unshare/unpin pkvm_pv_param page	ANDROID: pkvm: x86: Share/pin and unshare/unpin pkvm_pv_param page
f21553a7b1e71d5e5ff2e8163659bd366532afd6	ANDROID: pkvm: x86: Check the percpu pv_param before using it	ANDROID: pkvm: x86: Check the percpu pv_param before using it
7543fccfd679d49c9c99109a81a34156af1c6780	ANDROID: pkvm: x86: Fix buffer overrun in vmexit dump without a VM	ANDROID: pkvm: x86: Fix buffer overrun in vmexit dump without a VM
3266e57a69be9e272c01081ca49757d6632c7649	ANDROID: pkvm: x86: Pass nent instead of buffer size when setting cpuid	ANDROID: pkvm: x86: Pass nent instead of buffer size when setting cpuid
2f21934044fa3b3a32d311e108eb4b520f997fc3	ANDROID: pkvm: x86: Acquire the default cpuid leaves into the per-cpu buffer	ANDROID: pkvm: x86: Acquire the default cpuid leaves into the per-cpu buffer
891be3de090e36a28a58afd18ec998304e7e18c2	ANDROID: pkvm: x86: Enforce cpuid for pVM	ANDROID: pkvm: x86: Enforce cpuid for pVM
32c0d644e330d4e5116a26401311ce6b3df0fe2f	ANDROID: pkvm: x86: Rename and move kvm_call_pkvm()	ANDROID: pkvm: x86: Rename and move kvm_call_pkvm()
0e08212821c266ddd67fa36c0324e227ea5521b4	ANDROID: pkvm: x86: Consolidate pKVM host hypercall interfaces	ANDROID: pkvm: x86: Consolidate pKVM host hypercall interfaces
54f7c96538fac99d758e21b22707b35004787777	ANDROID: pkvm: x86: trace: Don't allocate struct perf_data on stack	ANDROID: pkvm: x86: trace: Don't allocate struct perf_data on stack
c00fa43780a62d9dbfe60d3e24470cabd536321b	ANDROID: pkvm: x86: trace: Add tracing of host hypercalls	ANDROID: pkvm: x86: trace: Add tracing of host hypercalls
89b16de6aded146c1124d96ef3346dc7f28379cb	ANDROID: pkvm: x86: Enforce initial CR0/CR4/EFER/RFLAGS for pvmfw	ANDROID: pkvm: x86: Enforce initial CR0/CR4/EFER/RFLAGS for pvmfw
69bd71c587e1c07ace57ccfcb2b000575f489fd2	ANDROID: pkvm: x86: Enforce initial values of most MSRs	ANDROID: pkvm: x86: Enforce initial values of most MSRs
e8081e2fe46b292adb5798e6ee5d9a4242c7305a	ANDROID: pkvm: x86: Prevent re-running vCPU after a triple fault	ANDROID: pkvm: x86: Prevent re-running vCPU after a triple fault
0634e456b7cb3333d9e6c1edbe63a146870e55bb	ANDROID: pkvm: x86: Reset vCPU when creating it	ANDROID: pkvm: x86: Reset vCPU when creating it
1692c669371d797fe3094f5b6183227614e2d05b	ANDROID: pkvm: x86: Share/unshare and pin/unpin vmexit perf buffer	ANDROID: pkvm: x86: Share/unshare and pin/unpin vmexit perf buffer
30de8cc37f65daf7ae5b367f7557b2b956d2e732	ANDROID: pkvm: x86: Acquire pv_param in irq disabled context	ANDROID: pkvm: x86: Acquire pv_param in irq disabled context
711b9174c8b92a47fced2ae2395bfce8091c49df	ANDROID: pkvm: x86: Adapt pv_param framework for all pkvm hypercalls	ANDROID: pkvm: x86: Adapt pv_param framework for all pkvm hypercalls
f4ebde6cab5dd1ff7f74bb63eee467b1587349c2	ANDROID: pkvm: x86: Fix the qi initialization logic with !IRQ_REMAP	ANDROID: pkvm: x86: Fix the qi initialization logic with !IRQ_REMAP
b37c89bc4af3aa63148052f3f1dfd094dc11aecb	ANDROID: pkvm: x86: Queued invalidation support requirement	ANDROID: pkvm: x86: Queued invalidation support requirement
6a1be1b061d698df9634eed5d3fdd021f1153819	ANDROID: pkvm: x86: intel_iommu_sm flag in pkvm	ANDROID: pkvm: x86: intel_iommu_sm flag in pkvm
500f4df2e1f9b2ca35a1455730b4c8682903cbfb	ANDROID: pkvm: x86: Initialize agaw and gaw fields in intel_iommu	ANDROID: pkvm: x86: Initialize agaw and gaw fields in intel_iommu
0d1fab6ea4c3279212c65bd85968153c79cd0783	ANDROID: pkvm: x86: Allow SRTP only once for simplicity	ANDROID: pkvm: x86: Allow SRTP only once for simplicity
1fe9ff0dcfee2f5230a3831c77931d540a104aa9	ANDROID: pkvm: x86: Read only sharing of pages with host	ANDROID: pkvm: x86: Read only sharing of pages with host
9a802b37ec7f75d881e15863de93b0afc0215a56	ANDROID: pkvm: x86: Factor out shadow iommu implementation	ANDROID: pkvm: x86: Factor out shadow iommu implementation
7f18b1c234f9ee2edd5e848d75180114bb3a798a	ANDROID: pkvm: x86: Introduce CONFIG_PKVM_INTEL_PVIOMMU	ANDROID: pkvm: x86: Introduce CONFIG_PKVM_INTEL_PVIOMMU
124be4e60392bfb8ba08571b1c5688dbe99a64ef	ANDROID: pkvm: x86: Expose intel_iommu_superpage to pkvm	ANDROID: pkvm: x86: Expose intel_iommu_superpage to pkvm
91d744415695af73d5c5c92f6a76b126d45c54ca	ANDROID: pkvm: x86: Drop support for hardware requiring write buffer flush	ANDROID: pkvm: x86: Drop support for hardware requiring write buffer flush
5e74ea8499f6f3f658d9c4c0193086ea9a21f3f1	ANDROID: pkvm: x86: Disable nested feature absent warning for pvIOMMU	ANDROID: pkvm: x86: Disable nested feature absent warning for pvIOMMU
47a804aee09a58e84eb3c88443a71a1d1beb725e	ANDROID: pkvm: x86: Don't reserve memory for pvIOMMU	ANDROID: pkvm: x86: Don't reserve memory for pvIOMMU
5638905558203ee0f7b7eaa154a4dec9d2166c10	ANDROID: pkvm: x86: Don't map page 0 in the hypervisor address space	ANDROID: pkvm: x86: Don't map page 0 in the hypervisor address space
ca91c8871871f368da0a0644e85ff78d9feeed40	ANDROID: pkvm: x86: Enforce secure startup of secondary vCPUs	ANDROID: pkvm: x86: Enforce secure startup of secondary vCPUs
01b35f214285f1b5c1214a2122886337ec79f7a5	ANDROID: SQUASHME: pkvm: x86: Prevent starting same vCPU twice	ANDROID: SQUASHME: pkvm: x86: Prevent starting same vCPU twice
80de493924fa6ccd6f5135b86e01201a9e0f7181	ANDROID: pkvm: x86: Use local copy of pv_param	ANDROID: pkvm: x86: Use local copy of pv_param
f8c16d49ea1f0368b31bf1458abe30d24c2953d7	ANDROID: pkvm: x86: Forbid host IQT MMIO access and add IEC flush hypercall	ANDROID: pkvm: x86: Forbid host IQT MMIO access and add IEC flush hypercall
02e065e39ba57adc1fa82efc6a6e09523fa81aa2	ANDROID: pkvm: x86: Hypercalls for legacy mode context tables	ANDROID: pkvm: x86: Hypercalls for legacy mode context tables
c36ccb7ba16d64299d012ddd1b0c973465a90d11	ANDROID: pkvm: x86: Hypercalls for scalable mode context update.	ANDROID: pkvm: x86: Hypercalls for scalable mode context update.
74194a840f31c53b197867e845dfd8cde69e0beb	ANDROID: pkvm: x86: PASID Table update hypercalls	ANDROID: pkvm: x86: PASID Table update hypercalls
ce4f0f4cf23d9fffcd3eeab3e2eb52325de1f69f	ANDROID: pkvm: x86: Return pasid dir and tables back to host	ANDROID: pkvm: x86: Return pasid dir and tables back to host
7815ebd0dfa128dfd1e35485781a156752c44f2c	ANDROID: pkvm: x86: Flush did FLPT_DEFAULT_DID for host_ept flush	ANDROID: pkvm: x86: Flush did FLPT_DEFAULT_DID for host_ept flush
8bfb49e77407cc434f04a4d3e988b00936cd68e9	ANDROID: pkvm: x86: Introduce iommu domain framework	ANDROID: pkvm: x86: Introduce iommu domain framework
409eb47a914307a5f409cde8fb5d49aa5fc4d4d3	ANDROID: pkvm: x86: Hypercalls for iommu domain management	ANDROID: pkvm: x86: Hypercalls for iommu domain management
cf073afacf0274634c2709fc0e6da425d14b795a	ANDROID: pkvm: x86: Move cache flushing logic into pkvm	ANDROID: pkvm: x86: Move cache flushing logic into pkvm
80977caf7436e2d179d2d6b2da57595f03a1b9a5	ANDROID: pkvm: x86: Move cache tag assign/unassign to pkvm	ANDROID: pkvm: x86: Move cache tag assign/unassign to pkvm
26593e8480fff1780c6e741ddc0ff6ed933a3f7e	ANDROID: pkvm: x86: Refuse context clear if active PASIDs exist	ANDROID: pkvm: x86: Refuse context clear if active PASIDs exist
0d10b8a4fcee2efa65714735bab5e983a9c38fab	ANDROID: pkvm: x86: Flag to denote if iotlb sync needed after map	ANDROID: pkvm: x86: Flag to denote if iotlb sync needed after map
aa92610c1b9a2a451dfab51e11dae84dda9e097f	ANDROID: iommu/vt-d: Move DOMAIN_MAX_PFN and DOMAIN_MAX_ADDR to header	ANDROID: iommu/vt-d: Move DOMAIN_MAX_PFN and DOMAIN_MAX_ADDR to header
22ade54aa7270e166ccb5afb53c658abf6e9c648	ANDROID: pkvm: x86: Hypercalls for domain pagetable management	ANDROID: pkvm: x86: Hypercalls for domain pagetable management
98eb4df6dd75f54c22ca4778544cf18df3650b37	ANDROID: pkvm: x86: Implement iommu map/unmap in pKVM	ANDROID: pkvm: x86: Implement iommu map/unmap in pKVM
8f126c7728cf694eed7518b5bcedc6a09945586e	ANDROID: pkvm: x86: Memory APIs to pin/unpin host owned pages for DMA	ANDROID: pkvm: x86: Memory APIs to pin/unpin host owned pages for DMA
e37217e7f399bd1a2a18336dbbe819fccc4ab3fe	ANDROID: pkvm: x86: Pin/unpin physical pages on DMA map/unmap	ANDROID: pkvm: x86: Pin/unpin physical pages on DMA map/unmap
408d43a43d4a1d2f52b18b30a7342aa1c1cedbe0	ANDROID: pkvm: x86: Use hardware ignored bit 63 to represent mapped PTE	ANDROID: pkvm: x86: Use hardware ignored bit 63 to represent mapped PTE
dd8e9f72096664a4150b187d9327ebdd13cfccdb	ANDROID: pkvm: x86: Check for mappings when switching to superpage	ANDROID: pkvm: x86: Check for mappings when switching to superpage
7ae8058846a914d756ac492c4ce70cb5e9b9c92c	pkvm: x86: Fix compiling issue with CONFIG_PKVM_INTEL_PVIOMMU=n	pkvm: x86: Fix compiling issue with CONFIG_PKVM_INTEL_PVIOMMU=n
529beaade4c310848a024ad8c81657efefbfaf6c	pkvm: x86: Fix compiling issue in pv_iommu.c	pkvm: x86: Fix compiling issue in pv_iommu.c



# 总结
1. **host os移到vmx non-root**：建立intel pkvm架构，让linux host从vmx root到vmx non-root，pkvm留在root mode，实际控制vmx/ept/iommu

d5b286f5a4e4 — pkvm: x86: Introduce pkvm_host_deprivilege_cpus
facc2107e6f8 — pkvm: x86: Deprivilege host OS
fb2971c97ec5 — pkvm: vmx: Reshuffle the host CPU deprivilege process

2. **把pkvm隔离**：将pKVM的代码和数据放到一个独立的section，启动时为它预留内存，并建立early allocator、buddy allocator、pKVM自己的页表和host EPT，pKVM runtime使用独立的VA/PA转换接口，并通过符号前缀限制对host kernel符号的依赖，让hypervisor在运行时不再依赖host kernel的地址空间和内存管理

3282557218ed — pkvm: x86: Build pKVM runtime as an independent binary
11a21ef399dd — Isolate pKVM & host
d057cc4d3956 — pkvm: x86: Create host EPT pgtable in init-finalise hypercall


3. **把VMX/KVM控制面下沉**：先做vmx/vmcs/ept emulation，让host KVM还能工作，后面逐步改成pv的接口，让vCPU的操作create/free/load/run、VM-exit、APIC、MSR、interrupt、MMU都被pKVM控制

6c3e04d07e93 — VMX emulation
70f4d72d6f73 — pkvm: vmx: Implement vcpu_run


4. **内存从shadow EPT变成PV MMU**：早期用shadow ept和page-state记录page ownership，pVM改用PV MMU hypercall，让pkvm来管理guest os的内存

c19bb3527ce6 — Memory protection based on page state
76011ef693d0 — pkvm: x86: Introduce shadow EPT
8ed244930343 — ANDROID: pkvm: x86: Switch to using PV MMU instead of emulated EPT


5. **完善pVM生命周期**：加入protected VM类型，补充pkvm的创建和销毁的过程，区分普通kvm和pkvm vm，pkvm来维护shadow vm/vcpu状态，启动的时候走安全启动；运行时由pkvm初始寄存器，就是补充pvm的创建到启动到运行到终止都由pkvm监控

00a33eb6ef71 — pkvm: x86: Rename KVM_X86_PROTECTED_VM to KVM_X86_PKVM_PROTECTED_VM
854828a784ef — pkvm: x86: Add hypercalls for shadow_vm/vcpu init & teardown
e8081e2fe46b — ANDROID: pkvm: x86: Prevent re-running vCPU after a triple fault


6. **增强IOMMU/DMA隔离**：把设备DMA交给pkvm管,最先是pKVM接管IOMMU的关键配置和MMIO访问，自己维护shadow IOMMU；后面改成PV IOMMU，host要改IOMMU context、PASID、domain、页表，或者做DMA map/unmap，都要通过pkvm hypercall

d1e7cd11e8c3 — pkvm: x86: init: isolate IOMMU from host VM
03644b199d72 — pkvm: x86: shadow IOMMU page table
98eb4df6dd75 — ANDROID: pkvm: x86: Implement iommu map/unmap in pKVM


7. **设备直通**：支持设备直通，但是不能让host完全控制。host可以保留直接io访问，保证系统一样；但如果设备要直通给pvm，pkvm必须知道这个设备是谁，并检查它的pci配置、IOMMU映射和DMA内存，确认不会越界访问别的VM或pKVM的内存

756f4fbc7da3 — pkvm: x86: Introduce pkvm_ptdev structure
ca911169d1ed — pkvm: x86: Implement the ADD_PTDEV hypercall
a019a98e6eab — pkvm: x86: Sync IOMMU page table for the attached ptdev
239cd97d6031 — pkvm: audit config space access for ptdev



pv 半虚拟化
ept and pv mmu
pkvm保护的IOMMU 的对象
pkvm设备直通的具体实现

host 访问 vpcu状态
host 访问 guest内存
host 访问 直通的设备(安全性 如果是host创建的一个virt device?)
