**本频道只提供STM32-HAL库编程方法，不提供知识点注解**

# 版权信息

© 2024 . 未经许可不得复制、修改或分发。 此文献为 [小風的藏書閣](https://t.me/xfp2333) 所有。

# SDIO使用FATFS系统编程指南

## FATFS 简介

**FATFS** 是一个轻量级的嵌入式文件系统，支持 FAT12、FAT16 和 FAT32 格式。它广泛应用于 SD 卡、USB 闪存等存储设备，具有小巧、可移植、高效的特点。

主要功能包括文件操作（如 `f_open`、`f_read`、`f_write`）、目录管理（如 `f_mkdir`、`f_opendir`）、文件系统挂载和格式化等。其架构分为用户接口层、文件系统层和存储设备接口层，需用户实现底层硬件驱动（如 SD 卡的 SPI 或 SDIO 通信）。

FATFS 常用于数据记录、固件更新和文件传输等场景，通过灵活配置可满足嵌入式系统的多种存储需求。

# Cubemx 配置

1. 配置SDIO模块 参考 [SDIO编程指南](./SDIO.MD)

2. 打开`MiddleWare and SofterWare Packs`选项卡，选择`FATFS`

3. 勾选`SD Card`选项

4. `Configuration`选项中配置`SetDefines`参数

| **Function Parameters配置**         | 描述                                                   |  
|---------------------------------------|-------------------------------------------------------|  
| **FS_READONLY (read only mode)**      | 只读模式开关。启用后，所有写操作将被禁止，文件系统只读。 |  
| **FS_MINIMIZE (Minimization level)**  | 最小化级别，用于减小代码大小，禁用未使用的功能。取值范围为 0 到 3，对应不同功能限制。 |  
| **USE_STRFUNC (String functions)**    | 字符串操作功能开关。启用后，可使用字符串相关函数，如 `f_gets` 等。 |  
| **USE_FIND (Find functions)**         | 查找功能开关。启用后，可使用 `f_findfirst` 和 `f_findnext` 进行文件查找。 |  
| **USE_MKFS (Make filesystem function)** | 文件系统格式化功能开关。启用后，可使用 `f_mkfs` 函数创建文件系统。 |  
| **USE_FASTSEEK (Fast seek function)** | 快速定位功能开关。启用后，可使用快速文件指针定位功能，加速大文件处理。 |  
| **USE_EXPAND (Use f_expand function)** | 文件扩展功能开关。启用后，可使用 `f_expand` 函数为文件预分配存储空间。 |  
| **USE_CHMOD (Change attributes)**     | 修改文件属性功能开关。启用后，可使用 `f_chmod` 和 `f_utime` 更改文件属性和时间戳。 |  
| **USE_FORWARD (Forward function)**    | 数据转发功能开关。启用后，可使用 `f_forward` 将数据直接转发到指定设备或处理函数。 |  

---

| **Locale and Namespace Parameters配置** | 描述 |  
|------------------------------------------|-------------------------------------------------------|  
| **CODE_PAGE (Code page on target)**       | 配置目标系统的代码页，用于支持特定语言的字符集编码，如 ASCII、UTF-8 等。 |  
| **USE_LFN (Use long filename)**           | 是否启用长文件名支持。启用后，可使用超过 8.3 格式的文件名，需占用更多内存。 |  
| **MAX_LFN (Max long filename)**           | 配置长文件名的最大长度，单位为字符，仅在启用长文件名支持时有效。 |  
| **LFN_UNICODE (Enable Unicode)**          | 启用 Unicode 编码支持，允许使用宽字符编码处理文件名。 |  
| **STRF_ENCODE (Character encoding)**      | 配置字符串函数的字符编码方式，如 UTF-8、UTF-16 等。 |  
| **FS_RPATH (Relative Path)**              | 启用相对路径支持，允许在文件操作中使用相对路径（如 `../`）。 |  


--- 

| **Physical Drive Parameters配置**      | 描述 |  
|------------------------------------------|-------------------------------------------------------|  
| **VOLUMES (Logical drives)**             | 配置逻辑驱动器的最大数量，决定支持的卷数。 |  
| **MAX_SS (Maximum Sector Size)**         | 配置支持的最大扇区大小，以字节为单位，如 512、1024 等。 |  
| **MIN_SS (Minimum Sector Size)**         | 配置支持的最小扇区大小，以字节为单位，如 512。 |  
| **MULTI_PARTITION (Volume partitions feature)** | 启用多分区支持，允许在一个物理驱动器上使用多个逻辑分区。 |  
| **USE_TRIM (Erase feature)**             | 启用 TRIM 擦除功能，用于提高支持 TRIM 的存储设备性能。 |  
| **FS_NOFSINFO (Force full FAT scan)**     | 禁用文件系统信息字段，强制完全扫描 FAT 表以提高兼容性。 |  

---

| **System Parameters配置**      | 描述 |  
|--------------------------------|-------------------------------------------------------|  
| **FS_TINY (Tiny mode)**        | 启用 Tiny 模式，减少内存占用，通过每次访问文件时动态分配文件表。 |  
| **FS_EXFAT (Support of exFAT file system)** | 启用 exFAT 文件系统支持，用于处理大容量存储设备。 |  
| **FS_NORTC (Timestamp feature)** | 如果没有实时时钟（RTC），禁用时间戳功能，所有文件的时间戳默认为固定值。 |  
| **FS_REENTRANT (Re-Entrancy)**  | 启用线程安全支持，允许多线程环境中使用文件系统。 |  
| **FS_TIMEOUT (Timeout ticks)**  | 配置文件系统的超时周期，用于线程同步的等待超时。 |  
| **FS_LOCK (Number of files opened simultaneously)** | 配置同时打开文件的最大数量，用于管理文件句柄锁定。 |  


5. `Configuration`选项中配置`Advanced Settings`参数

- 在`Advanced Settings`配置页面中，参数 `SDIO instance` 只可选`SDIO`，无SPI选项，因为STM32F407的SDIO接口不支持SPI；当SDIO配置了`DMA`时，如果希望FatFs使用SDIO的DMA，那么可以将参数 `Use dma template` 配置为`Enable`；参数 `BSP code for SD` 不可选，只能为通用模式


## FatFs DMA读写SD卡流程简述

1. 在`SDIO`配置界面找到`DMA Setting`添加`SDIO_RX`和`SDIO_TX`通道即可

2. 找到`FATFS`选项的`Advanced Settings`选项卡

- 使能`Use dma template`选项

3. 打开`SDIO`的全局中断

# API部分

- **`FATFS`简介**  
  `FATFS` 是一个独立于硬件的平台无关的文件系统库，不属于 STM32 的 HAL 库。我们可以通过移植，将其应用到 STM32 开发中，支持各种嵌入式存储设备。

- **参考资源**  
  - [FATFS官网](http://elm-chan.org/fsw/ff/archives.html)：可以下载 FATFS 源码及查看官方文档。

- **移植说明**  
  如果用户希望将 FATFS 手动移植到自己的嵌入式系统，需要关注以下内容：  

  1. **关键文件**  
     - `diskio.c` 和 `diskio.h` 是 FATFS 移植的核心文件。
  
  2. **底层函数实现**  
     根据存储设备类型（如 RAM、MMC、USB）实现以下底层函数：  


| **函数名**       | **功能描述**                              |  
|------------------|------------------------------------------|  
| `disk_status()`   | 获取存储设备的当前状态。                   |  
| `disk_initialize()` | 初始化存储设备。                          |  
| `disk_read()`     | 从存储设备读取数据块。                      |  
| `disk_write()`    | 向存储设备写入数据块。                      |  
| `disk_ioctl()`    | 执行存储设备的 I/O 控制操作。               |  


  3. **上层接口**  
     一旦完成以上底层函数的实现，即可使用 FATFS 提供的上层接口（如 `f_open()`）操作底层存储设备，实现文件操作功能（如读写文件、目录管理等）。  
- 使用cubeMX不用这么麻烦

 4. **在 FatFs 中，大多数的API都拥有一个名为 FRESULT 的结构体返回值，其包含了20个枚举对象**

 ```c
typedef enum {
	FR_OK = 0,				/* (0) 成功 */
	FR_DISK_ERR,			/* (1) 在Disk IO层发生硬错误，检查user_diskio.c中代码 */
	FR_INT_ERR,				/* (2) 参数检查错误 */
	FR_NOT_READY,			/* (3) 物理驱动器不工作 */
	FR_NO_FILE,				/* (4) 找不到文件 */
	FR_NO_PATH,				/* (5) 找不到路径 */
	FR_INVALID_NAME,		/* (6) 路径名称格式无效，检查是否8.3格式/是否支持长文件名 */
	FR_DENIED,				/* (7) 因禁止访问或目录满导致无法访问 */
	FR_EXIST,				/* (8) 因禁止访问导致无法访问 */
	FR_INVALID_OBJECT,		/* (9) 文件/目录无效 */
	FR_WRITE_PROTECTED,		/* (10) 物理驱动器写保护 */
	FR_INVALID_DRIVE,		/* (11) 逻辑驱动器号无效 */
	FR_NOT_ENABLED,			/* (12) 卷无工作区 */
	FR_NO_FILESYSTEM,		/* (13) 无有效FAT卷 */
	FR_MKFS_ABORTED,		/* (14) 函数f_mkfs()因为问题终止 */
	FR_TIMEOUT,				/* (15) 不能在限定时间内获得访问卷的许可 */
	FR_LOCKED,				/* (16) 因为文件共享策略导致操作被拒绝 */
	FR_NOT_ENOUGH_CORE,		/* (17) 不能分配长文件名工作缓存区 */
	FR_TOO_MANY_OPEN_FILES,	/* (18) 打开文件个数大于_FS_LOCK */
	FR_INVALID_PARAMETER	/* (19) 无效参数 */
} FRESULT;


- 函数的定义与文件都是要自己添加的

 ```
file_operate.h文件
```c
#ifndef FILE_OPERATE_H
#define FILE_OPERATE_H
 
#include "main.h"
#include "FatFs.h"
#include "stdio.h"
 
/*定义自己的存储设备*/
/*用户存储设备扇区字节数*/
#define User_Sector 4096
/*用户存储设备FatFS对象*/
#define User_FatFs 	USERFatFS
/*用户存储设备卷路径*/
#define User_SDPath USERPath
/*用户存储设备初始化类型*/
#define User_FatType FM_FAT
 
/*函数声明*/
void Mount_FatFs(void);
void FatFs_GetDiskInfo(void);
void FatFs_ScanDir(const TCHAR* PathName);
void FatFs_ReadTXTFile(TCHAR *filename);
void FatFs_WriteTXTFile(TCHAR *filename,uint16_t year, uint8_t month, uint8_t day);
void FatFs_GetFileInfo(TCHAR *filename);
void FatFs_DeleteFile(TCHAR *filename);
void FatFs_PrintfFileDate(WORD date, WORD time);
 
#endif
```

file_operate.c文件
```c
#include "file_operate.h"
 
//定义用于格式化的工作区缓存
BYTE workBuffer[4*User_Sector];
 
/*挂载FatFs文件系统*/
void Mount_FatFs(void)
{
	//挂载文件系统
	FRESULT retUSER = f_mount(&User_FatFs, User_SDPath, 1);
	//发生错误
	if(retUSER != FR_OK)
	{
		//没有文件系统，需要格式化
		if(retUSER == FR_NO_FILESYSTEM)
		{
			printf("\r\n没有文件系统,开始格式化\r\n");
			//创建文件系统
			retUSER = f_mkfs(User_SDPath, User_FatType, 0, workBuffer, 4*User_Sector);
			//格式化失败
			if(retUSER != FR_OK)
			{
				printf("格式化失败，错误代码 = %d\r\n", retUSER);
			}
			//格式化成功
			else
			{
				printf("格式化成功，开始重新挂载\r\n");
				//有文件系统后重新挂载
				retUSER = f_mount(&User_FatFs, User_SDPath, 1);
				//挂载失败
				if(retUSER != FR_OK)
				{
					printf("发生错误，错误代码 = %d\r\n", retUSER);
				}
				//挂载成功
				else
				{
					printf("*** 文件系统挂载成功 ***\r\n");
				}
			}
		}
		//不是没有文件系统，而是发生其他错误
		else
		{
			printf("发生其他错误，错误代码 = %d\r\n", retUSER);
		}
	}
	//有文件系统直接挂在成功
	else
	{
		printf("文件系统挂载成功\r\n");
	}
}
 
/*获取磁盘信息并在LCD上显示*/
void FatFs_GetDiskInfo(void)
{
    FATFS *fs;
	//定义剩余簇个数变量
    DWORD fre_clust; 
	//获取剩余簇个数
    FRESULT res = f_getfree("0:", &fre_clust, &fs); 
	//获取失败
    if(res != FR_OK)
    {
        printf("f_getfree() error\r\n");
        return;
    }
    printf("\r\n*** FAT disk info ***\r\n");
		
	//总的扇区个数
    DWORD tot_sect = (fs->n_fatent - 2) * fs->csize;  
		
	//剩余的扇区个数 = 剩余簇个数 * 每个簇的扇区个数
    DWORD fre_sect = fre_clust * fs->csize;    
		
	//对于SD卡和U盘, _MIN_SS=512字节
#if  _MAX_SS == _MIN_SS  
    //SD卡的_MIN_SS固定为512，右移11位相当于除以2048
	//剩余空间大小，单位：MB，用于SD卡，U盘
    DWORD freespace= (fre_sect>>11); 
		//总空间大小，单位：MB，用于SD卡，U盘		
    DWORD totalSpace= (tot_sect>>11);  
#else
	//Flash存储器，小容量
	//剩余空间大小，单位：KB
    DWORD freespace= (fre_sect*fs->ssize)>>10;   
	//总空间大小，单位：KB
    DWORD totalSpace= (tot_sect*fs->ssize)>>10;  
#endif
 
	//FAT类型
    printf("FAT type = %d\r\n",fs->fs_type);
    printf("[1=FAT12,2=FAT16,3=FAT32,4=exFAT]\r\n");
		
	//扇区大小，单位字节
    printf("Sector size(bytes) = ");
	//SD卡固定512字节
#if  _MAX_SS == _MIN_SS 
    printf("%d\r\n", _MIN_SS);
#else
	//FLASH存储器
    printf("%d\r\n", fs->ssize);
#endif
		
    printf("Cluster size(sectors) = %d\r\n", fs->csize);
    printf("Total cluster count = %ld\r\n", fs->n_fatent-2);
    printf("Total sector count = %ld\r\n", tot_sect);
		
	//总空间
#if  _MAX_SS == _MIN_SS 
    printf("Total space(MB) = %ld\r\n", totalSpace);
#else
    printf("Total space(KB) = %ld\r\n", totalSpace);
#endif
		
	//空闲簇数量
    printf("Free cluster count = %ld\r\n",fre_clust);
	//空闲扇区数量
    printf("Free sector count = %ld\r\n", fre_sect);
		
	//空闲空间
#if  _MAX_SS == _MIN_SS 
    printf("Free space(MB) = %ld\r\n", freespace);
#else
    printf("Free space(KB) = %ld\r\n", freespace);
#endif
 
    printf("Get FAT disk info OK\r\n");
}
 
/*创建文本文件*/
void FatFs_WriteTXTFile(TCHAR *filename,uint16_t year, uint8_t month, uint8_t day)
{
	FIL	file;
	printf("\r\n*** Creating TXT file: %s ***\r\n", filename);
	
	FRESULT res = f_open(&file, filename, FA_CREATE_ALWAYS | FA_WRITE);
	//打开/创建文件成功
	if(res == FR_OK)
	{
		//字符串必须有换行符"\n"
		TCHAR str[]="Line1: Hello, FatFs***\n";  
		//不会写入结束符"\0"
		f_puts(str, &file); 
		
		printf("Write file OK: %s\r\n", filename);
	}
	else
	{
		printf("Open file error，error code: %d\r\n", res);
	}
	//使用完毕关闭文件
	f_close(&file);
}
 
/*读取一个文本文件的内容*/
void FatFs_ReadTXTFile(TCHAR *filename)
{
	printf("\r\n*** Reading TXT file: %s ***\r\n", filename);
 
	FIL	file;
	//以只读方式打开文件
	FRESULT res = f_open(&file, filename, FA_READ);  
	//打开成功
	if(res == FR_OK)
	{
		//读取缓存
		TCHAR str[100];
		//没有读到文件内容末尾
		while(!f_eof(&file))
		{
			//读取1个字符串,自动加上结束符”\0”
			f_gets(str,100, &file);	
			printf("%s", str);
		}
		printf("\r\n");
	}
	//如果没有该文件
	else if(res == FR_NO_FILE)
		printf("File does not exist\r\n");
	//打开失败
	else
		printf("f_open() error，error code: %d\r\n", res);
	//关闭文件
	f_close(&file);
}
 
/*扫描和显示指定目录下的文件和目录*/
void FatFs_ScanDir(const TCHAR* PathName)
{
	DIR dir;					//目录对象
	FILINFO fno;				//文件信息
	//打开目录
	FRESULT res = f_opendir(&dir, PathName);
	//打开失败
	if(res != FR_OK)
	{
		//关闭目录，直接退出函数
		f_closedir(&dir);
		printf("\r\nf_opendir() error，error code: %d\r\n", res);
		return;
	}
	
	printf("\r\n*** All entries in dir: %s ***\r\n", PathName);
	//顺序读取目录中的文件
	while(1)
	{
		//读取目录下的一个项
		res = f_readdir(&dir, &fno);    
		//文件名为空表示没有多的项可读了
		if(res != FR_OK || fno.fname[0] == 0)
			break;  
		//如果是一个目录
		if(fno.fattrib & AM_DIR)  		
		{
			printf("DIR: %s\r\n", fno.fname);
		}
		//如果是一个文件
		else  		
		{
			printf("FILE: %s\r\n",fno.fname);
		}
	}
	//扫描完毕，关闭目录
	printf("Scan dir OK\r\n");
	f_closedir(&dir);
}
 
/*获取一个文件的文件信息*/
void FatFs_GetFileInfo(TCHAR *filename)
{
	printf("\r\n*** File info of: %s ***\r\n", filename);
 
	FILINFO fno;
	//检查文件或子目录是否存在
	FRESULT fr = f_stat(filename, &fno);
	//如果存在从fno中读取文件信息
	if(fr == FR_OK)
	{
		printf("File size(bytes) = %ld\r\n", fno.fsize);
		printf("File attribute = 0x%x\r\n", fno.fattrib);
		printf("File Name = %s\r\n", fno.fname);
		//输出创建/修改文件时的时间戳
		FatFs_PrintfFileDate(fno.fdate, fno.ftime);
	}
	//如果没有该文件
	else if (fr == FR_NO_FILE)
		printf("File does not exist\r\n");
	//发生其他错误
	else
		printf("f_stat() error，error code: %d\r\n", fr);
}
 
/*删除文件*/
void FatFs_DeleteFile(TCHAR *filename)
{
	printf("\r\n*** Delete File: %s ***\r\n", filename);
	FIL	file;
	//打开文件
	FRESULT res = f_open(&file, filename, FA_OPEN_EXISTING);  
	if(res == FR_OK)
	{
		//关闭文件
		f_close(&file);
		printf("open successfully!\r\n");
	}
	//删除文件
	res = f_unlink(filename);
	//删除成功
	if(res == FR_OK)
	{
		printf("The file was deleted successfully!\r\n");
	}
	//删除失败
	else
	{
		printf("File deletion failed, error code：%d\r\n", res);
	}
}
 
/*打印输出文件日期*/
void FatFs_PrintfFileDate(WORD date, WORD time)
{
	printf("File data = %d/%d/%d\r\n", ((date>>9)&0x7F)+1980, (date>>5)&0xF, date&0x1F);
	printf("File time = %d:%d:%d\r\n", (time>>11)&0x1F, (time>>5)&0x3F, time&0x1F);
}
 
```


## 使用示例

```c
/*main.c中添加头文件*/
#include "file_operate.h" 
 
/*外输初始化完进入主循环前*/
//检测SPI与W25Q128通信是否正常
printf("Reset,ID:0x%x\r\n", Flash_ReadID());
//挂载文件系统
Mount_FatFs();
//获取磁盘信息
FatFs_GetDiskInfo();
 
/*主循环中按键逻辑*/
/*按键WK_UP被按下*/
if(HAL_GPIO_ReadPin(WK_UP_GPIO_Port,WK_UP_Pin) == GPIO_PIN_SET)
{
	HAL_Delay(50);
	if(HAL_GPIO_ReadPin(WK_UP_GPIO_Port,WK_UP_Pin) == GPIO_PIN_SET)
	{
		FatFs_ScanDir("0:/");
		while(HAL_GPIO_ReadPin(WK_UP_GPIO_Port,WK_UP_Pin));
	}
}
 
/*按键KEY2被按下*/
if(HAL_GPIO_ReadPin(KEY2_GPIO_Port,KEY2_Pin) == GPIO_PIN_RESET)
{
	HAL_Delay(50);
	if(HAL_GPIO_ReadPin(KEY2_GPIO_Port,KEY2_Pin) == GPIO_PIN_RESET)
	{
		FatFs_WriteTXTFile("test.txt",2016,11,15);
		while(!HAL_GPIO_ReadPin(KEY2_GPIO_Port,KEY2_Pin));
	}
}
 
/*按键KEY1被按下*/
if(HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin) == GPIO_PIN_RESET)
{
	HAL_Delay(50);
	if(HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin) == GPIO_PIN_RESET)
	{
		FatFs_ReadTXTFile("test.txt");
		FatFs_GetFileInfo("test.txt");
		while(!HAL_GPIO_ReadPin(KEY1_GPIO_Port,KEY1_Pin));
	}
}
 
/*按键KEY0被按下*/
if(HAL_GPIO_ReadPin(KEY0_GPIO_Port,KEY0_Pin) == GPIO_PIN_RESET)
{
	HAL_Delay(50);
	if(HAL_GPIO_ReadPin(KEY0_GPIO_Port,KEY0_Pin) == GPIO_PIN_RESET)
	{
		FatFs_DeleteFile("test.txt");
		while(!HAL_GPIO_ReadPin(KEY0_GPIO_Port,KEY0_Pin));
	}
}
```