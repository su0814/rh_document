文件下载指令，此指令无CMD_TYPE,而是属于一个公共的子指令，其可以被各个CMD_TYPE所共用。



文件下载指令仅有CMD和SUB,其定义如下：

【CMD】

```c
enum {
    CMD_PUBLIC_FILE_DOWNLOAD = 0X80,
};  // public CMD option
```

【SUB】

```c
enum {
    SUB_PUBLIC_FILE_DOWNLOAD_SOH = 0x01,  // start of data packet
    SUB_PUBLIC_FILE_DOWNLOAD_STX = 0x02,  // Data packet transfer
    SUB_PUBLIC_FILE_DOWNLOAD_EOT = 0x03,  // End of transmission

    SUB_PUBLIC_FILE_DOWNLOAD_SOH_ACK = 0x10,
    SUB_PUBLIC_FILE_DOWNLOAD_STX_ACK = 0x11,
    SUB_PUBLIC_FILE_DOWNLOAD_EOT_ACK = 0x12,

};  // CMD_PUBLIC_FILE_DOWNLOAD option
```

