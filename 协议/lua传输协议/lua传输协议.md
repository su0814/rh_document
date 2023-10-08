[TOC]



## 1. transport CRC 协议数据包

本文提到的所有数据包在发送、接收时，都遵循以下格式：

| 帧起始     | ID   | CMD_TYPE | CMD    | SUB    | DATA_LEN           | DATA               | CRC   | 帧结尾     |
| ---------- | ---- | -------- | ------ | ------ | ------------------ | ------------------ | ----- | ---------- |
| 0xAA, 0xAA | 0x00 | 1 Byte   | 1 Byte | 1 Byte | 2 Byte, 低字节在前 | 长度由前一字段定义 | 2字节 | 0x55, 0x55 |

其中：

- CRC校验的是从“id”字段开始到“data”字段的内容

- 打包时，除了增加0xAA和0x55的帧头尾和crc校验字段以外，如果从“ID”开始到“data”结束，还存在0xAA、0x55或0xA5数据，则在其前面增加一个0xA5作为转义字符

- **【ID集】**

  | ID   | 备注                             |
  | ---- | -------------------------------- |
  | 00   | 公共ID，所有板卡都会相应此ID指令 |
  | 01   | ID_A                             |
  | 02   | ID_B                             |

  

## 2 命令集定义

### 2.1. 命令类型枚举(CMD_TYPE)

```c
enum {
    CMD_TYPE_BL    = 0x20,
    CMD_TYPE_LUA   = 0X21,//用于LUA的指令
};
```

"CMD_TYPE_LUA"的命令集

```c
enum {
    CMD_LUA_CALL		    = 0X00,//LUA的呼叫服务指令
    CMD_LUA_REPORT			= 0X01,//LUA的信息上报
};
enum {
    CMD_PUBLIC_FILE_DOWNLOAD = 0X80,
}; 
```

"CMD_PUBLIC_FILE_DOWNLOAD"的子命令集

```c
enum {
    SUB_PUBLIC_FILE_DOWNLOAD_SOH = 0x01,  // start of data packet
    SUB_PUBLIC_FILE_DOWNLOAD_STX = 0x02,  // Data packet transfer
    SUB_PUBLIC_FILE_DOWNLOAD_EOT = 0x03,  // End of transmission

    SUB_PUBLIC_FILE_DOWNLOAD_SOH_ACK = 0x10,
    SUB_PUBLIC_FILE_DOWNLOAD_STX_ACK = 0x11,
    SUB_PUBLIC_FILE_DOWNLOAD_EOT_ACK = 0x12,

}; 
```

## 3.文件下载指令

### 3.1 头包

上位机下发头包指令：

| ID         | CMD_TYPE     | CMD                      | SUB                          | LEN                                | DATA |
| ---------- | ------------ | ------------------------ | ---------------------------- | ---------------------------------- | ---- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_PUBLIC_FILE_DOWNLOAD | SUB_PUBLIC_FILE_DOWNLOAD_SOH | 1+filename_len+4(2byte,低字节在前) |      |

【DATA定义】

| 文件名长度        | 文件名                     | 文件大小 |
| ----------------- | -------------------------- | -------- |
| 1字节(最大64字节) | 不定长，跟随实际文件名长度 | 4字节    |

MCU回复ack：

**【SOH故障码】**

| 故障码 | 故障定义     |
| ------ | ------------ |
| 00     | 无故障       |
| -1     | 文件名过长   |
| -2     | 文件大小不符 |

| ID         | CMD_TYPE     | CMD                      | SUB                              | LEN  | DATA            |
| ---------- | ------------ | ------------------------ | -------------------------------- | ---- | --------------- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_PUBLIC_FILE_DOWNLOAD | SUB_PUBLIC_FILE_DOWNLOAD_SOH_ACK | 1    | 见【SOH故障码】 |

### 3.2 数据包

上位机下发数据包指令：

一包携带的最大文件数据量为1024字节，若文件大小超过512字节，则进行分包发送，否则根据文件实际大小发送

| ID         | CMD_TYPE     | CMD                      | SUB                          | LEN        | DATA                                |
| ---------- | ------------ | ------------------------ | ---------------------------- | ---------- | ----------------------------------- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_PUBLIC_FILE_DOWNLOAD | SUB_PUBLIC_FILE_DOWNLOAD_STX | 1+data_len | 包序号(4字节)+文件数据(最大512byte) |

MCU回复ack：

**【STX故障码】**

| 故障码 | 故障定义                      |
| ------ | ----------------------------- |
| 00     | 无故障                        |
| -1     | 包序号不对，有丢包现象        |
| -2     | 包携带文件数据量大小与len不符 |

| ID         | CMD_TYPE     | CMD                      | SUB                              | LEN  | DATA                                         |
| ---------- | ------------ | ------------------------ | -------------------------------- | ---- | -------------------------------------------- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_PUBLIC_FILE_DOWNLOAD | SUB_PUBLIC_FILE_DOWNLOAD_STX_ACK | 1    | 1字节【STX故障码】+4字节包序号（低字节在前） |

### 3.3 结束包

上位机下发结束包指令：

| ID         | CMD_TYPE     | CMD                      | SUB                          | LEN  | DATA      |
| ---------- | ------------ | ------------------------ | ---------------------------- | ---- | --------- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_PUBLIC_FILE_DOWNLOAD | SUB_PUBLIC_FILE_DOWNLOAD_EOT | 16   | 文件的MD5 |

MCU回复ack：

**【EOT故障码】**

| 故障码 | 故障定义                                                |
| ------ | ------------------------------------------------------- |
| 00     | 无故障                                                  |
| -1     | SOH下发的包的数据大小与实际接收到的数据大小不符，有丢包 |
| -2     | MD5校验失败                                             |

| ID         | CMD_TYPE     | CMD                      | SUB                              | LEN  | DATA            |
| ---------- | ------------ | ------------------------ | -------------------------------- | ---- | --------------- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_PUBLIC_FILE_DOWNLOAD | SUB_PUBLIC_FILE_DOWNLOAD_EOT_ACK | 1    | 见【EOT故障码】 |

## 4.呼叫服务指令

上位机下发调用指令：

【指令码】

| 指令码 | 指令定义 | 参数 |
| ------ | -------- | ---- |
| 00     | run      | NONE |
| 01     | stop     | NONE |
| 02     | log      |      |



| ID         | CMD_TYPE     | CMD          | SUB  | LEN                  | DATA        |
| ---------- | ------------ | ------------ | ---- | -------------------- | ----------- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_LUA_CALL | 0    | 两个byte，低字节在前 | 指令码(u16) |

MCU回复ack：

**【CALL故障码】**

| 故障码 | 故障定义     |
| ------ | ------------ |
| 00     | 指令收到     |
| -1     | 指令不存在   |
| -2     | 指令执行失败 |

| ID         | CMD_TYPE     | CMD          | SUB  | LEN  | DATA             |
| ---------- | ------------ | ------------ | ---- | ---- | ---------------- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_LUA_CALL | 00   | 1    | 见【CALL故障码】 |

## 5.信息上报指令

针对MCU主动上报的信息，提供一个上报通道：

MCU主动上报：

| ID         | CMD_TYPE     | CMD            | SUB  | LEN                  | DATA                  |
| ---------- | ------------ | -------------- | ---- | -------------------- | --------------------- |
| 见【ID集】 | CMD_TYPE_LUA | CMD_LUA_REPORT | 00   | 两个byte，低字节在前 | 上报内容(ASCLL码格式) |

## 5.信息上报指令