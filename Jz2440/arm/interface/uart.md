# UART 使用
```
为了调试方便，调试uart，使其能够打印uart信息
```

![](image/uart.png)

## 1. fifo模式与非fifo模式
```
  1. 从上述图片中可知，非fifo的数据缓冲区只是fifo模式缓冲区中的一个字节
  2. fifo模式与非fifo模式的区别也就是触发中断的时机
    fifo模式：发送/接收的数据字节数达到fifo深度，则触发相应的中断
    非fifo模式：发送/接收一个数据字节，则触发相应的中断
  3. fifo模式下，UTXH0是发送fifo0的入口地址
     fifo模式下，URXH0是接收fifo0的入口地址
```

## 2. 配置uart

### 2.1 涉及到的串口寄存器
```
  本配置采用非fifo模式，同时不使用流控
  简单配置一下串口：
  1. 涉及到的uart0寄存器
    UCON0 ：UART 控制寄存器
    ULCON0 ：UART 线路控制寄存器
    UBRDIV0：UART 波特率分频寄存器
    UTXH0：UART 发送缓冲寄存器
    URXH0：UART 接收缓冲寄存器
```

### 2.2 代码展示
```
void uart0_init()
{
    GPHCON &= ~((3<<4) | (3<<6));   
	GPHCON |= ((2<<4) | (2<<6));
//将IO端口配置成uart0模式

	GPHUP &= ~((1<<2) | (1<<3));  
//将IO端口配置成上拉模块（起始信号是从高电平到低电平，所以默认是高电平）

    UCON0 = 0x5;  //选择PCLK作为波特率的时钟源，使能发送与接收功能  


    ULCON0 = 0x3;  //配置帧参数为 8n1
    UBRDIV0 =  26; //配置波特率参数，PCLK=50M

}
int putchar(int c)
{
    while (!(UTRSTAT0 & (1<<2)));
	UTXH0 = (unsigned char)c;
    return 0;

}

char getchar()
{
    while (!(UTRSTAT0 & (1<<0)));
	return URXH0;
}

int puts(const char *s)
{
	while (*s)
	{
		putchar(*s);
		s++;
	}
	return 0;
}

int main(void)
{
	unsigned char c;
	uart0_init();                //初始化串口
	puts("Hello, world!\n\r");   //打印串口数据
	while(1)
	{
		c = getchar();
		if (c == '\r')
		{
			putchar('\n');
		}
		if (c == '\n')
		{
			putchar('\r');
		}
		putchar(c);
	}
	return 0;
}

```
