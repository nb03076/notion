# 정리

```c
#include <linux/interrupt.h>

1.
0이면 정상적으로 등록됨
if(request_irq(...)) {
	 에러 처리 코드
}

irq 번호는 unsigned int로 정의

2. smp_processor_id() : cpu id 알려줌

3. 
local_irq_save(flags);
pr_info("flags:%02lx\n", flags);
local_irq_restore(flags);
결과 : flags:246

4.
irqs_disabled() 1이면 irq disable 된 상태 0이면 enable

5.
gpio_is_valid(gpio_led) 1이면 존재 0이면 존재x
return -ENODEV

gpio_request(gpio_led, "my_led") 0이면 성공
gpio_free(gpio_led)

gpio_direction_output(gpio_led, 0);
gpio_set_value(gpio_led, 1);

gpio_set_debounce(gpio_button, 1000); 디바운스 제거
irq번호 = gpio_to_irq(gpio 번호)

6.
struct task_struct* tsk = kthread_run(스레드함수, 파라미터, "스레드 이름")
while(!kthread_should_stop()) {
}
kthread_stop(tsk);

7.
0이면 성공
request_threaded_irq(irq, 
			my_interrupt, my_threaded_interrupt,
			IRQF_SHARED, "my_interrupt", &my_dev_id));
핸들러에서 IRQ_WAKE_THREAD 해야지 irq 스레드 동작함

8.
local_softirq_pending() 비트맵에 softirq 펜딩 알려줌 

9.
tasklet_init(이름,함수,dev) 이나 DECLARE_TASKLET(이름,함수)
tasklet_schedule(&my_tasklet);
void tasklet_function(struct tasklet_struct *h) {

}

10.
tasklet count가 0이면 동작하고 그외면 동작안함

11.
워크
struct work_struct work;
INIT_WORK(&work, work_fn);
queue_work(system_wq, &work); //schdule_work하면 system_wq로 고정
static void work_fn(struct work_struct *work)

실제로는
if (queue_work(system_wq, &work))
		pr_info("work queued\n");

if (cancel_work_sync(&deferred_work.work))
		pr_info("work cancelled\n");

if (flush_work(&deferred_work2.work))
		pr_info("work2 flushed\n");

12.
딜레이 워크
struct delayed_work work;
static void work_fn(struct work_struct *work)
INIT_DELAYED_WORK(&work, work_fn);
schedule_delayed_work(&work, msecs_to_jiffies(5000));
혹은
queue_delayed_work(my_queue, &deferred_work1.work, msecs_to_jiffies(1000));

cancel_delayed_work(&work);
flush_delayed_work(&work);

13.
static void work_fn(struct work_struct *work)
이 핸들러 내부에서
struct delayed_work *my_delayed_work = to_delayed_work(work); 이렇게 사용가능
이거는 container_of 사용한 함수임

struct workqueue_struct *my_queue = NULL;
my_queue = alloc_workqueue("my_queue", WQ_UNBOUND, 1);
queue_work(my_queue, &work);

exit에서
flush_workqueue(my_queue);
destroy_workqueue(my_queue);
```