# 메모리 allocation

- MMU : 가상 주소를 물리적 주소로 변환
- 논리주소는 항상 가상주소

page는 PAGE_SIZE 라는 고정길이 가짐

페이지 프레임 : RAM의 고정길이 블록 참조

Page Frame Number(PFN)

페이지 테이블 : 가상, 물리적 주소간의 매핑 저장

**CONFIG_PAGE_OFFSET = 0xC0000000**

가상주소가 주어지면 이게 유저영역인지 커널 영역인지 알수 있다.

**Early in the boot process, the kernel permanently maps that 896 MB onto physical RAM. Addresses that result from that mapping are called logical addresses.**

논리주소 : 물리적 주소에 선형적으로 매핑된 커널공간의 주소

가상주소 : 항상 동일한 선형 1:1 매핑 가지진 않음(커널 공간주소에서 물리적 주소로 매핑되는건 동일)

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled.png)

**ZONE_DMA : 16MB 미만, 페이지 프레임**

**ZONE_NORMAL : 16~896mb 페이지 프레임**

**ZONE_HIGHMEM : 페이지 프레임. 근데 이건 64비트나 아니면 메모리 크기가 작으면 사용안함. 보통 어플리케이션에서 주로 사용함(주소 공간 명시적 매핑해야해서)**

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%201.png)

```c
이게 task_struct에 내장되어있음
메모리 매핑 테이블
struct mm_struct(include/linux/mm_types.h)

struct mm_struct {
    struct vm_area_struct *mmap; (프로세스영역 VMA저장)
    unsigned long mmap_base;
    unsigned long task_size;
    unsigned long highest_vm_end;
    pgd_t * pgd;
    atomic_t mm_users;
    atomic_t mm_count;
    atomic_long_t nr_ptes;
#if CONFIG_PGTABLE_LEVELS > 2
    atomic_long_t nr_pmds;
#endif
    int map_count;
    spinlock_t page_table_lock;
    unsigned long total_vm;
    unsigned long locked_vm;
    unsigned long pinned_vm;
    unsigned long data_vm;
    unsigned long exec_vm;
    unsigned long stack_vm;
    unsigned long start_code, end_code, start_data, end_data;
    unsigned long start_brk, brk, start_stack;
    unsigned long arg_start, arg_end, env_start, env_end;
    /* ref to file /proc/<pid>/exe symlink points to */
    struct file __rcu *exe_file;
};

start_code는 text segment 시작주소

struct vm_area_struct {
    unsigned long vm_start; 
    unsigned long vm_end;
    struct vm_area_struct *vm_next, *vm_prev;
    struct mm_struct *vm_mm;
    pgprot_t vm_page_prot;
    unsigned long vm_flags;
    unsigned long vm_pgoff;
    struct file * vm_file;
    [...]
}

vm_start : vma내 첫 주소
vm_end : vma외부 첫 주소
vm_mm : vma가 속한 프로세스 주소 공간
vm_page_protvm_flags : 액세스 권한
vm_file : 매핑 지원하는 파일
vm_pgoffvm_file : 페이지 크기 단위 오프셋(페이지수)
```

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%202.png)

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%203.png)

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%204.png)

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%205.png)

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%206.png)

## SLAB

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%207.png)

**`GFP_USER`: 사용자 메모리 할당용.**

**`GFP_KERNEL`: 커널 할당에 일반적으로 사용되는 플래그입니다.**

**`GFP_HIGHMEM`: `HIGH_MEM`영역에서 메모리를 요청합니다.**

**`GFP_ATOMIC`: 잠들 수 없는 원자적 방식으로 메모리를 할당합니다. 인터럽트 컨텍스트에서 메모리를 할당해야 할 때 사용됩니다.**

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%208.png)

![Untitled](%E1%84%86%E1%85%A6%E1%84%86%E1%85%A9%E1%84%85%E1%85%B5%20allocation%20b0238cd5277342e7bbb3cfb39f997f63/Untitled%209.png)

**<linux/slab.h>**

**void *kmalloc(size_t size, int flags);**

```c
GFP_KERNEL: 표준 플래그입니다. 코드가 잠들 수 있기 때문에 인터럽트 핸들러에서 이 플래그를 사용할 수 없습니다. 항상 LOM_MEM영역(따라서 논리 주소)에서 메모리를 반환합니다.
GFP_ATOMIC: 이것은 할당의 원자성을 보장합니다. 이 플래그는 인터럽트 컨텍스트에서 할당이 필요할 때 사용됩니다. 메모리는 비상 풀 또는 메모리에서 할당되므로 사용을 남용해서는 안 됩니다.
GFP_USER: 사용자 공간 프로세스에 메모리를 할당합니다. 그런 다음 메모리가 구별되고 분리됩니다.커널에 할당된 것에서.
GFP_NOWAIT: 예를 들어 인터럽트 핸들러 사용과 같이 원자적 컨텍스트 내에서 할당이 수행되는 경우에 사용됩니다. 이 플래그는 할당을 수행하는 동안 직접 회수, I/O 및 파일 시스템 작업을 방지합니다. 와 달리 GFP_ATOMIC메모리 예약을 사용하지 않습니다. 결과적으로 메모리 부족 상태에서는 GFP_NOWAIT할당이 실패할 가능성이 높습니다.
GFP_NOIO: 와 마찬가지로 GFP_USER차단할 수 있지만 와 달리 GFP_USER디스크 I/O를 시작하지 않습니다. 즉, 할당을 수행하는 동안 모든 I/O 작업을 방지합니다. 이 플래그는 주로 블록/디스크 계층에서 사용됩니다.
GFP_NOFS: 직접 회수를 사용하지만 파일 시스템 인터페이스는 사용하지 않습니다.
__GFP_NOFAIL: 호출자가 할당 실패를 처리할 수 없기 때문에 가상 메모리 구현은 무기한 재시도해야 합니다. 할당은 무기한 지연될 수 있지만 절대 실패하지 않습니다. 결과적으로 실패를 테스트하는 것은 쓸모가 없습니다.
GFP_HIGHUSER: HIGH_MEMORY영역에서 메모리 할당을 요청합니다.
GFP_DMA: 에서 메모리를 할당합니다 DMA_ZONE.
```