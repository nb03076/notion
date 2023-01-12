# input 장치

```c
#include <linux/input.h>

struct input_dev {
  const char *name;
  const char *phys; // 물리 경로
  unsigned long evbit[BITS_TO_LONGS(EV_CNT)]; // 이벤트 비트맵
// EV_KEY : 키 EV_REL : 상대위치(마우스) EV_ABS : 절대위치(조이스틱)
// set_bit(EV_KEY, my_input_dev->evbit);
  unsigned long keybit[BITS_TO_LONGS(KEY_CNT)]; // BTN_0 등...
  unsigned long relbit[BITS_TO_LONGS(REL_CNT)]; // REL_X 등..
  unsigned long absbit[BITS_TO_LONGS(ABS_CNT)]; // ABS_Y 등..
  unsigned long mscbit[BITS_TO_LONGS(MSC_CNT)]; // misc.. 기타
  unsigned int repeat_key; // 이전에 눌러진 키를 가지고있음
  int rep[REP_CNT]; // 
  struct input_absinfo *absinfo; // 절대축의 정보
// void input_set_abs_params(struct input_dev *dev,
//                       unsigned int axis, int min,
//                       int max, int fuzz, int flat)
  unsigned long key[BITS_TO_LONGS(KEY_CNT)]; // 현재 상태
  int (*open)(struct input_dev *dev);
  void (*close)(struct input_dev *dev);
  unsigned int users;
  struct device dev;
  unsigned int num_vals;
  unsigned int max_vals;
  struct input_value *vals;
  bool devres_managed;
};
```

```c
장치 등록
struct input_dev *input_allocate_device(void)
struct input_dev *devm_input_allocate_device(
                                       struct device *dev)
void input_free_device(struct input_dev *dev)
int input_register_device(struct input_dev *dev)
void input_unregister_device(struct input_dev *dev)
```

## 예시

```c
struct input_dev *idev;
int error;
/*
 * such allocation will take care of memory freeing and
 * device unregistering
 */
idev = devm_input_allocate_device(&client->dev);
if (!idev)
    return -ENOMEM;
idev->name = BMA150_DRIVER;
idev->phys = BMA150_DRIVER "/input0";
idev->id.bustype = BUS_I2C;
idev->dev.parent = &client->dev;
set_bit(EV_ABS, idev->evbit);
input_set_abs_params(idev, ABS_X, ABSMIN_ACC_VAL,
                     ABSMAX_ACC_VAL, 0, 0);
input_set_abs_params(idev, ABS_Y, ABSMIN_ACC_VAL,
                     ABSMAX_ACC_VAL, 0, 0);
input_set_abs_params(idev, ABS_Z, ABSMIN_ACC_VAL,
                     ABSMAX_ACC_VAL, 0, 0);
error = input_register_device(idev);
if (error)
    return error;
error = devm_request_threaded_irq(&client->dev,
            client->irq,
            NULL, my_irq_thread,
            IRQF_TRIGGER_RISING | IRQF_ONESHOT,
            BMA150_DRIVER, NULL);
if (error) {
    dev_err(&client->dev, "irq request failed %d, 
           error %d\n", client->irq, error);
    return error;
}
return 0;
```

## input-polldev는 5버전? 부터 사라진 기능이라 분석안함