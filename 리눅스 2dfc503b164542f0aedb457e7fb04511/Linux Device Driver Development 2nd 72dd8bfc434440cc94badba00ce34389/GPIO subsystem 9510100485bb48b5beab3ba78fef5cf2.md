# GPIO subsystem

```c
#include <linux/pinctrl/pinconf.h>
#include <linux/pinctrl/pinconf-generic.h>
#include <linux/pinctrl/pinctrl.h>
#include <linux/pinctrl/pinmux.h>

struct pinctrl_desc {
     const char *name;
     const struct pinctrl_pin_desc *pins;
     unsigned int npins; // ARRAY_SIZE 로 표기
     const struct pinctrl_ops *pctlops; // 옵션.. 없어도 되기는 함
     const struct pinmux_ops *pmxops;
     const struct pinconf_ops *confops;
     struct module *owner;
[...]
};

struct pinctrl_pin_desc {
     unsigned number;
     const char *name;
[...]
};

struct pinctrl_dev *devm_pinctrl_register(
                      struct device *dev,
                      struct pinctrl_desc *pctldesc,
                      void *driver_data);

#include <linux/pinctrl/consumer.h>
```

```c
&usdhc4 {
[...]
     pinctrl-0 = <&pinctrl_usdhc4_1>;
     pinctrl-names = "default";
};

iomux 노드에서 핀 정의한거 &pinctrl_usdhc4_1 한것...

핀 그룹 선택하고 configure 하려면 아래처럼 해줘야함
#include <linux/pinctrl/consumer.h>
int ret;
struct pinctrl_state *s;
struct pinctrl *p;
foo_probe()
{
    p = devm_pinctrl_get(dev);
    if (IS_ERR(p))
        return PTR_ERR(p);
    s = pinctrl_lookup_state(p, name);
    if (IS_ERR(s))
        return PTR_ERR(s);
    ret = pinctrl_select_state(p, s);
    if (ret < 0) // on error
        return ret;
[...]
}
```

```c
#include <linux/gpio/driver.h>

struct gpio_chip {
     const char       *label;
     struct gpio_device    *gpiodev;
     struct device         *parent;
     struct module         *owner;
     int        (*request)(struct gpio_chip *gc,
                           unsigned int offset);
     void       (*free)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*get_direction)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*direction_input)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*direction_output)(struct gpio_chip *gc,
                      unsigned int offset, int value);
     int       (*get)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*get_multiple)(struct gpio_chip *gc,
                           unsigned long *mask,
                           unsigned long *bits);
     void       (*set)(struct gpio_chip *gc,
                      unsigned int offset, int value);
     void       (*set_multiple)(struct gpio_chip *gc,
                           unsigned long *mask,
                           unsigned long *bits);
     int        (*set_config)(struct gpio_chip *gc,
                            unsigned int offset,
                            unsigned long config);
     int        (*to_irq)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*init_valid_mask)(struct gpio_chip *gc,
                              unsigned long *valid_mask,
                              unsigned int ngpios);
// 유효한 gpio 비트 마스크
     int       (*add_pin_ranges)(struct gpio_chip *gc);
     int        base; // gpiochip에서 처리하는 첫 번째 GPIO 번호. 음수이면 커널이 동적으로 할당
     u16        ngpio; // base + ngpio - 1
     const char *const *names;
     bool       can_sleep;
#if IS_ENABLED(CONFIG_GPIO_GENERIC)
     unsigned long (*read_reg)(void __iomem *reg);
     void (*write_reg)(void __iomem *reg, unsigned long data);
     bool be_bits;
     void __iomem *reg_dat;
     void __iomem *reg_set;
     void __iomem *reg_clr;
     void __iomem *reg_dir_out;
     void __iomem *reg_dir_in;
     bool bgpio_dir_unreadable;
     int bgpio_bits;
     spinlock_t bgpio_lock;
     unsigned long bgpio_data;
     unsigned long bgpio_dir;
#endif /* CONFIG_GPIO_GENERIC */
#ifdef CONFIG_GPIOLIB_IRQCHIP
     struct gpio_irq_chip irq;
#endif /* CONFIG_GPIOLIB_IRQCHIP */
     unsigned long *valid_mask;
#if defined(CONFIG_OF_GPIO)
     struct device_node *of_node;
     unsigned int of_gpio_n_cells;
     int (*of_xlate)(struct gpio_chip *gc,
                const struct of_phandle_args *gpiospec,
                u32 *flags);
#endif /* CONFIG_OF_GPIO */
};

int devm_gpiochip_add_data(struct device *dev,
                        struct gpio_chip *gc, void *data)
```

## mcp23016 예

```c
#include <linux/gpio.h>

#define GPIO_NUM 16
struct mcp23016 {
    struct i2c_client *client;
    struct gpio_chip gpiochip;
    struct mutex lock;
};
static int mcp23016_probe(struct i2c_client *client)
{
  struct mcp23016 *mcp;
  
  if (!i2c_check_functionality(client->adapter,
      I2C_FUNC_SMBUS_BYTE_DATA))
    return -EIO;
  mcp = devm_kzalloc(&client->dev, sizeof(*mcp),
                      GFP_KERNEL);
  if (!mcp)
    return -ENOMEM;
  mcp->gpiochip.label = client->name;
  mcp->gpiochip.base = -1;
  mcp->gpiochip.dev = &client->dev;
  mcp->gpiochip.owner = THIS_MODULE;
  mcp->gpiochip.ngpio = GPIO_NUM; /* 16 */
  /* may not be accessed from atomic context */
  mcp->gpiochip.can_sleep = 1; 
  mcp->gpiochip.get = mcp23016_get_value;
  mcp->gpiochip.set = mcp23016_set_value;
  mcp->gpiochip.direction_output =
                          mcp23016_direction_output;
  mcp->gpiochip.direction_input =
                          mcp23016_direction_input;
  mcp->client = client;
  i2c_set_clientdata(client, mcp);
  return devm_gpiochip_add_data(&client->dev,
                                &mcp->gpiochip, mcp);
}
```

```c
struct gpio_irq_chip {
     struct irq_chip *chip;
     struct irq_domain *domain;
     const struct irq_domain_ops *domain_ops;
     irq_flow_handler_t handler;
     unsigned int default_type;
     irq_flow_handler_t parent_handler;
     union {
          void *parent_handler_data;
          void **parent_handler_data_array;
     };
     unsigned int num_parents;
     unsigned int *parents;
     unsigned int *map;
     bool threaded;
     bool per_parent_data;
     int (*init_hw)(struct gpio_chip *gc);
     void (*init_valid_mask)(struct gpio_chip *gc,
                      unsigned long *valid_mask,
                      unsigned int ngpios);
     unsigned long *valid_mask;
     unsigned int first;
     void       (*irq_enable)(struct irq_data *data);
     void       (*irq_disable)(struct irq_data *data);
     void       (*irq_unmask)(struct irq_data *data);
     void       (*irq_mask)(struct irq_data *data);
};

ex)

static struct irq_chip mcp23016_irq_chip = {
     .name = "gpio-mcp23016",
     .irq_mask = mcp23016_irq_mask,
     .irq_unmask = mcp23016_irq_unmask,
     .irq_set_type = mcp23016_irq_set_type,
};

static int mcp23016_probe(struct i2c_client *client)
{
    struct gpio_irq_chip *girq;
    struct irq_chip *irqc;
[...]
    girq = &mcp->gpiochip.irq;
    girq->chip = &mcp23016_irq_chip;
    /* This will let us handling the parent IRQ in the driver */
    girq->parent_handler = NULL;
    girq->num_parents = 0;
    girq->parents = NULL;
    girq->default_type = IRQ_TYPE_NONE;
    girq->handler = handle_level_irq;
    girq->threaded = true;
[...]
    /*
     * Directly request the irq here instead of passing
     * a flow-handler.
     */
    err = devm_request_threaded_irq(
                      &client->dev,
                      client->irq,
                      NULL, mcp23016_irq,
                      IRQF_TRIGGER_RISING | IRQF_ONESHOT,
                      dev_name(&i2c->dev), mcp);
[...]
    
    return devm_gpiochip_add_data(&client->dev,
                                &mcp->gpiochip, mcp);
}

static irqreturn_t mcp23016_irq(int irq, void *data)
{
    struct mcp23016 *mcp = data;
    unsigned long status, changes, child_irq, i;
    status = read_gpio_status(mcp);
    mutex_lock(&mcp->lock);
    change = mcp->status ^ status;
    mcp->status = status;
    mutex_unlock(&mcp->lock);
    /* Do some stuff, may be adapting "change" according to level */
    [...]
    for_each_set_bit(i, &change, mcp->gpiochip.ngpio) {
        child_irq =
           irq_find_mapping(mcp->gpiochip.irq.domain, i);
        handle_nested_irq(child_irq);
    }
    return IRQ_HANDLED;
}
```

```c
#include <linux/gpio/consumer.h>

struct gpio_desc {
     struct gpio_chip *chip;
     unsigned long    flags;
     const char       *label;
};

struct gpio_desc *gpiod_get_index(struct device *dev,
                                 const char *con_id,
                                 unsigned int idx,
                                 enum gpiod_flags flags)
struct gpio_desc *gpiod_get(struct device *dev,
                            const char *con_id,
                            enum gpiod_flags flags)
struct gpio_desc *gpiod_get_optional(struct device *dev,
                                     const char *con_id,
                                     enum gpiod_flags flags);

enum gpiod_flags {
    GPIOD_ASIS  = 0,
    GPIOD_IN = GPIOD_FLAGS_BIT_DIR_SET,
    GPIOD_OUT_LOW = GPIOD_FLAGS_BIT_DIR_SET |
                    GPIOD_FLAGS_BIT_DIR_OUT,
    GPIOD_OUT_HIGH = GPIOD_FLAGS_BIT_DIR_SET |
                     GPIOD_FLAGS_BIT_DIR_OUT |
                     GPIOD_FLAGS_BIT_DIR_VAL,
};

/* 방향 변경 */
int gpiod_direction_input(struct gpio_desc *desc);
int gpiod_direction_output(struct gpio_desc *desc,
                           int value);

/* sleep 가능 여부 */
int gpiod_cansleep(const struct gpio_desc *desc);

int gpiod_get_value_cansleep(const struct gpio_desc *desc);
void gpiod_set_value_cansleep(struct gpio_desc *desc,
                              int value);

int gpiod_get_value(const struct gpio_desc *desc);
void gpiod_set_value(struct gpio_desc *desc, int value);

int gpiod_to_irq(const struct gpio_desc *desc);

void gpiod_put(struct gpio_desc *desc);
```

```c
시스템 프로그래밍에서 사용하는 gpiod 라이브러리는 일단 분석안했음
```