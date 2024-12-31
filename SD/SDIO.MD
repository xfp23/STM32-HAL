
**本频道只提供STM32-HAL库编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

# STM32 SDIO编程指南

- STM32F407提供了一个SDIO接口可以直接通过HAL库来驱动`1/4位`总线宽度的SD卡或`1/4/8`位总线宽度的多媒体卡，其完全兼容SD卡规范版本`2.0`，但只支持高速SD卡，也即与SD卡进行数据传输最大速度为`25MHz`

- SDIO由`APB`2接口和`SDIO适配器`两部分组成，SDIO适配器提供了驱动`SD/MMC`卡的全部功能，APB2接口则可以访问SDIO适配器寄存器在适当时候向内核发起中断/DMA请求

- SDIO适配器由`48MHz`的`SDIOCLK`驱动，根据`SDIOCLK时钟频率`、 `SDIO Clock divider bypass` 参数和 `SDIOCLK clock divide factor` 参数就可以确定与SD卡通信时SDIO_CLK的时钟频率，当时钟分频器旁路使能时，`SDIO_CLK=SDIOCLK`；当时钟分频器旁路不使能时，`SDIO_CLK=SDIOCLK / (2+时钟分频因子)`；

- 由于STM32F407的SDIO只支持高速SD卡，因此时钟分频器旁路常常不使能，这样`当时钟分频因子为0时，SDIO_CLK则达到最大速度48MHz / 2 = 24Mhz`，但在实际的使用中往往稍微降低该时钟频率，否则可能会出现读写SD卡失败的现象


- **SD卡初始化的时候应该以不超过400KHz的速率，1位总线宽度进行初始化，否则将初始化失败**

- **硬件设计时需要注意MCU与SD卡通信的1/4根数据线SDIO_D0/1/2/3和命令线SDIO_CMD均需外部上拉 (47k上拉电阻)**
[SD卡知识点扩展](./SDEX.MD)

## CubeMX配置

1. 选择告诉晶振、配置锁相环，飙到允许的最高频率

2. 找到SDIO硬件资源配置选择合适的模式


### SD/MMC 模式对比表

| 模式                  | 描述                                                                                 |  
|-----------------------|-------------------------------------------------------------------------------------|  
| **SD 1 bit**          | SD卡单线模式，数据线数量为1条，速度较慢，适用于低速场景。                             |  
| **SD 4 bit Wide bus** | SD卡多线模式，数据线数量为4条，速度较快，广泛用于嵌入式设备中。                        |  
| **MMC 1 bit**         | MMC卡单线模式，数据线数量为1条，兼容性高，但速度受限。                               |  
| **MMC 4 bit Wide bus**| MMC卡多线模式，数据线数量为4条，提供更高的数据传输速度。                              |  
| **MMC 8 bit Wide bus**| MMC卡多线模式，数据线数量为8条，适用于高速数据传输需求，但硬件设计更复杂。             |  


3. 打开`Parameter Settis`选项卡

### SDIO 配置选项表

| **选项**                                           | **描述**                                                                                          |  
|----------------------------------------------------|--------------------------------------------------------------------------------------------------|  
| **Clock transition on which the bit capture is made** | （时钟跳变沿捕获数据配置）：设置数据捕获边沿，可选择上升沿或下降沿。                                    |  
| **SDIO Clock divider bypass** | （时钟分频器旁路使能）：使能该参数时，SDIO_CLK=SDIOCLK；否则SDIO_CLK频率由时钟分频因子决定。                                  |  
| **SDIO Clock output enable when the bus is idle**  | （空闲模式时钟输出使能）：用于节能模式，控制总线空闲时是否输出时钟信号。                                 |  
| **SDIO hardware flow control**                    | （硬件流控）：设置是否启用 SDIO 的硬件流控，通常默认不启用以简化配置。                                 |  
| **SDIOCLK clock divide factor**                   | （时钟分频因子）：当不启用时钟分频器旁路时，SDIO_CLK 的频率为 SDIOCLK 除以（2+时钟分频因子）。             |  




## HAL库API

- API示例

### 1. **`HAL_SD_GetCardInfo`** 函数注释示例
```c
/**
 * @brief 获取 SD 卡的类型、版本、大小等信息，存储在 CardInfo 结构体中。
 * 
 * 该函数用于从 SD 卡中读取基本信息（例如卡类型、版本、卡片容量等），并将这些信息存储在 `CardInfo` 结构体中。
 * 
 * @param hsd  指向 SD 卡句柄的指针，该句柄在初始化时由 `HAL_SD_Init()` 函数提供。
 * @param CardInfo  指向一个 `HAL_SD_CardInfoTypeDef` 结构体的指针，用于存储从 SD 卡读取的详细信息。
 * 
 * @return 返回 `HAL_OK` 表示成功，返回其他值表示发生了错误（例如 `HAL_ERROR`）。
 */
HAL_StatusTypeDef HAL_SD_GetCardInfo(SD_HandleTypeDef *hsd, HAL_SD_CardInfoTypeDef *CardInfo);
```

### 2. **`HAL_SD_GetCardStatus`** 函数注释示例
```c
/**
 * @brief 获取 SD 卡的当前状态信息。
 * 
 * 该函数用于从 SD 卡中读取当前的状态信息（例如数据总线宽度、卡的擦除超时等），并将这些信息存储在 `CardStatus` 结构体中。
 * 
 * @param hsd  指向 SD 卡句柄的指针，该句柄在初始化时由 `HAL_SD_Init()` 函数提供。
 * @param CardStatus  指向一个 `HAL_SD_CardStatusTypeDef` 结构体的指针，用于存储从 SD 卡读取的状态信息。
 * 
 * @return 返回 `HAL_OK` 表示成功，返回其他值表示发生了错误（例如 `HAL_ERROR`）。
 */
HAL_StatusTypeDef HAL_SD_GetCardStatus(SD_HandleTypeDef *hsd, HAL_SD_CardStatusTypeDef *CardStatus);
```

### 3. **`HAL_SD_Erase`** 函数注释示例
```c
/**
 * @brief 擦除 SD 卡中的指定块范围。
 * 
 * 该函数用于擦除 SD 卡中从 `BlockAddrStart` 到 `BlockAddrEnd` 指定范围内的块。
 * 
 * @param hsd  指向 SD 卡句柄的指针，该句柄在初始化时由 `HAL_SD_Init()` 函数提供。
 * @param BlockAddrStart  指定起始块地址。
 * @param BlockAddrEnd  指定结束块地址。
 * 
 * @return 返回 `HAL_OK` 表示成功，返回其他值表示发生了错误（例如 `HAL_ERROR`）。
 */
HAL_StatusTypeDef HAL_SD_Erase(SD_HandleTypeDef *hsd, uint32_t BlockAddrStart, uint32_t BlockAddrEnd);
```

### 4. **`HAL_SD_WriteBlocks`** 函数注释示例
```c
/**
 * @brief 向 SD 卡中的指定块地址写入数据。
 * 
 * 该函数用于将数据从缓冲区 `pData` 写入 SD 卡的指定块地址 `BlockAddr`。
 * 
 * @param hsd  指向 SD 卡句柄的指针，该句柄在初始化时由 `HAL_SD_Init()` 函数提供。
 * @param pData  指向数据缓冲区的指针，数据将被写入到 SD 卡。
 * @param BlockAddr  写入数据的起始块地址。
 * @param BlockCount  要写入的块数。
 * @param TimeOut  写入操作的超时时间（单位为毫秒）。
 * 
 * @return 返回 `HAL_OK` 表示成功，返回其他值表示发生了错误（例如 `HAL_ERROR`）。
 */
HAL_StatusTypeDef HAL_SD_WriteBlocks(SD_HandleTypeDef *hsd, uint8_t *pData, uint32_t BlockAddr, uint32_t BlockCount, uint32_t TimeOut);
```

### 5. **`HAL_SD_ReadBlocks`** 函数注释示例
```c
/**
 * @brief 从 SD 卡中读取指定块地址的数据。
 * 
 * 该函数用于从 SD 卡的指定块地址 `BlockAddr` 读取数据，并将数据存储到缓冲区 `pData` 中。
 * 
 * @param hsd  指向 SD 卡句柄的指针，该句柄在初始化时由 `HAL_SD_Init()` 函数提供。
 * @param pData  指向数据缓冲区的指针，读取的数据将被存储到该缓冲区中。
 * @param BlockAddr  读取数据的起始块地址。
 * @param BlockCount  要读取的块数。
 * @param TimeOut  读取操作的超时时间（单位为毫秒）。
 * 
 * @return 返回 `HAL_OK` 表示成功，返回其他值表示发生了错误（例如 `HAL_ERROR`）。
 */
HAL_StatusTypeDef HAL_SD_ReadBlocks(SD_HandleTypeDef *hsd, uint8_t *pData, uint32_t BlockAddr, uint32_t BlockCount, uint32_t TimeOut);
```

- 程序示例

```c
/*显示SD卡的信息*/
void SDCard_ShowInfo(void)
{
	//SD卡信息结构体变量
	HAL_SD_CardInfoTypeDef cardInfo;  
	HAL_StatusTypeDef res = HAL_SD_GetCardInfo(&hsd, &cardInfo);
 
	if(res!=HAL_OK)
	{
		printf("HAL_SD_GetCardInfo() error\r\n");
		return;
	}
 
	printf("\r\n*** HAL_SD_GetCardInfo() info ***\r\n");
	printf("Card Type= %d\r\n", cardInfo.CardType);
	printf("Card Version= %d\r\n", cardInfo.CardVersion);
	printf("Card Class= %d\r\n", cardInfo.Class);
	printf("Relative Card Address= %d\r\n", cardInfo.RelCardAdd);
	printf("Block Count= %d\r\n", cardInfo.BlockNbr);
	printf("Block Size(Bytes)= %d\r\n", cardInfo.BlockSize);
	printf("LogiBlockCount= %d\r\n", cardInfo.LogBlockNbr);
	printf("LogiBlockSize(Bytes)= %d\r\n", cardInfo.LogBlockSize);
	printf("SD Card Capacity(MB)= %d\r\n", cardInfo.BlockNbr>>1>>10);
}
 
//获取SD卡当前状态
void SDCard_ShowStatus(void)
{
	//SD卡状态结构体变量
	HAL_SD_CardStatusTypeDef cardStatus;
	HAL_StatusTypeDef res = HAL_SD_GetCardStatus(&hsd, &cardStatus);
 
	if(res!=HAL_OK)
	{
		printf("HAL_SD_GetCardStatus() error\r\n");
		return;
	}
 
	printf("\r\n*** HAL_SD_GetCardStatus() info ***\r\n");
	printf("DataBusWidth= %d\r\n", cardStatus.DataBusWidth);
	printf("CardType= %d\r\n", cardStatus.CardType);
	printf("SpeedClass= %d\r\n", cardStatus.SpeedClass);
	printf("AllocationUnitSize= %d\r\n", cardStatus.AllocationUnitSize);
	printf("EraseSize= %d\r\n", cardStatus.EraseSize);
	printf("EraseTimeout= %d\r\n", cardStatus.EraseTimeout);
}
 
/*SD卡擦除测试*/
void SDCard_EraseBlocks(void)
{
	uint32_t BlockAddrStart=0;
	uint32_t BlockAddrEnd=10;
	
	printf("\r\n*** Erasing blocks ***\r\n");
 
	if(HAL_SD_Erase(&hsd, BlockAddrStart, BlockAddrEnd)==HAL_OK)
		printf("Erasing blocks,OK\r\n");
	else
		printf("Erasing blocks,fail\r\n");
 
	HAL_SD_CardStateTypeDef cardState=HAL_SD_GetCardState(&hsd);
	printf("GetCardState()= %d\r\n", cardState);
 
	while(cardState != HAL_SD_CARD_TRANSFER)
	{
		HAL_Delay(1);
		cardState=HAL_SD_GetCardState(&hsd);
	}
	printf("Blocks 0-10 is erased.\r\n");
}
 
/*SD卡写入测试函数*/
void SDCard_TestWrite(void)
{
	printf("\r\n*** Writing blocks ***\r\n");
	
	// BLOCKSIZE为512，在stm32f4xx_hal_sd.h中被定义
	uint8_t pData[BLOCKSIZE]="Hello, welcome to UPC\0";  
	uint32_t BlockAddr=5; 	
	uint32_t BlockCount=1; 
	uint32_t TimeOut=1000;	
 
	if(HAL_SD_WriteBlocks(&hsd,pData,BlockAddr,BlockCount,TimeOut) == HAL_OK)
	{
		printf("Write to block 5, OK\r\n");
		printf("The string is: %s\r\n", pData);
	}
	else
	{
		printf("Write to block 5, fail ***\r\n");
		return;
	}
 
	for(uint16_t i=0;i<BLOCKSIZE; i++)
		pData[i]=i; 	
 
	BlockAddr=6;
	if(HAL_SD_WriteBlocks(&hsd,pData,BlockAddr,BlockCount,TimeOut) == HAL_OK) 
	{
		printf("Write block 6, OK\r\n");
		printf("Data in [10:15] is: ");
		for (uint16_t j=11; j<=15;j++)
		{
			printf("%d,", pData[j]);
		}
		printf("\r\n");
	}
	else
		printf("Write to block 6, fail ***\r\n");
}
 
/*SD卡读取测试函数*/
void SDCard_TestRead(void)	
{
	printf("\r\n*** Reading blocks ***\r\n");
 
	uint8_t pData[BLOCKSIZE];
	uint32_t BlockAddr=5;
	uint32_t BlockCount=1;
	uint32_t TimeOut=1000;
 
	if(HAL_SD_ReadBlocks(&hsd,pData,BlockAddr,BlockCount,TimeOut) == HAL_OK)
	//if(HAL_SD_ReadBlocks_IT(&hsd,pData,BlockAddr,BlockCount) == HAL_OK)
	{
		printf("Read block 5, OK\r\n");
		printf("The string is: %s\r\n", pData);
	}
	else
	{
		printf("Read block 5, fail ***\r\n");
		return;
	}
 
	BlockAddr=6;
	if(HAL_SD_ReadBlocks(&hsd,pData,BlockAddr,BlockCount,TimeOut)== HAL_OK)
	//if(HAL_SD_ReadBlocks_IT(&hsd,pData,BlockAddr,BlockCount) == HAL_OK)
	{
		printf("Read block 6, OK\r\n");
		printf("Data in [10:15] is: ");
 
		for (uint16_t j=11; j<=15;j++)
		{
			printf("%d,", pData[j]);
		}
		printf("\r\n");
	}
}
 ```
# <span style="color:red;">重大错误</span>

目前遇到初始化过后芯片进入硬件错误循环，无法正常初始化，还没排查到原因。
