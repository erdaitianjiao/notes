### ebpf开发流程

##### CO-RE (Compile Once – Run Everywhere)

##### 一、hello_world环境搭建

1. 生成vmlinux.h

   ```bash
   bpftool btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h
   ```

2. 编写内核态代码 (hello.bpf.c)

   ```c
   #include <linux/bpf.h>
   #include <bpf/bpf_helpers.h>
   
   SEC("tp/syscalls/sys_enter_execve")
   int handle_tp(void *ctx)
   {
       unsigned long long pid = bpf_get_current_pid_tgid() >> 32;
       bpf_printk("hello from pid %u\n", pid);
       return 0;
   }
   
   char LICENSE[] SEC("license") = "GPL";
   ```

3. 编写用户态代码(hello.c)

   ```c
   #include <stdio.h>
   #include <unistd.h>
   #include <signal.h>
   #include "hello.skel.h"
   
   static volatile bool exiting = false;
   static void sig_handler(int sig) { exiting = true; }
   
   int main() {
       struct hello_bpf *skel;
       int err;
   
       // 1. 打开并加载 BPF 程序
       skel = hello_bpf__open_and_load();
       if (!skel) return 1;
   
       // 2. 附加到内核挂载点
       err = hello_bpf__attach(skel);
       if (err) goto cleanup;
   
       signal(SIGINT, sig_handler);
       printf("Successfully started! Run 'sudo cat /sys/kernel/debug/tracing/trace_pipe' to see output.\n");
   
       while (!exiting) {
           sleep(1);
       }
   
   cleanup:
       hello_bpf__destroy(skel);
       return 0;
   }
   ```

   

4. 编译

   ```bash
   # 编译bpf.c生成.o文件
   clang -g -O2 -target bpf -c hello.bpf.c -o hello.bpf.o
   # 生成骨架文件
   bpftool gen skeleton hello.bpf.o > hello.skel.h
   # 编译用户态程序生成可执行文件
   gcc -g -O2 hello.c -lbpf -o hello
   ```

   