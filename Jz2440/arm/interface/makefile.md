# 本文档介绍一个最简单的裸板代码及编译

```sh
  本文的作用如下：
    1. 如何设置栈指针
    2. 如何设置链接地址（相当于写一个简单的链接地址）
```

<div>cat led_on.S:</div>

```sh

.text
.global _start
_start:

//关闭看么狗
	ldr r0,=0x53000000
	ldr r1, =0
	str r1 ,[r0]

/* 设置内存: sp 栈 ：
 *     nand启动-->前4K是sram
 *      nor启动-->sram的起始地址是0x40000000
 *
 *
 * 分辨是nor/nand启动
 * 写0到0地址, 再读出来
 * 如果得到0, 表示0地址上的内容被修改了, 它对应ram, 这就是nand启动
 * 否则就是nor启动
*/

	mov r1 ,#0
	ldr r0 ,[r1]               //读出0地址的值放入到r0寄存器
	str r1,[r1]                //写0到0地址
	ldr r2 ,[r1]               //读出0地址的值放入到r2寄存器
	cmp r1,r2                  //比较r0与r2的值的大小
	ldr sp ,=0x40000000+4096   //默认为nor启动时的栈指针
	moveq sp, #4096            //如果r1与r2的值相等，则是nand启动
	streq r0 ,[r1]             //如果是nand启动，则还原0地址的值
	bl main                    //跳转到main函数执行


halt:                        //汇编定义一个死循环
	b halt
```


<div>led.c:</div>

```sh
int main()
{
	return 0;
}
```


<div>Makefile:</div>

```sh

#  $@ --->目标
#  $^ --->所有的依赖
#  $< --->第一个依赖

objs = led_on.o
objs += led.o

target = led

all: $(objs)
	arm-none-eabi-ld -Ttext 0 $^ -o $(target).elf
	arm-none-eabi-objcopy -O binary -S $(target).elf $(target).bin
	arm-none-eabi-objdump -D $(target).elf > $(target).dis

%.o:%.c
	arm-none-eabi-gcc -c -o $@ $<

%.o:%.S
	arm-none-eabi-gcc -c -o $@ $<
clean:
	rm *.bin *.o *.elf *.dis -rf
```
