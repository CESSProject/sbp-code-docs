## Overview

CESS-Bucket (hereinafter referred to as Bucket/bucket) is a member of the CESS (Cumulus Encrypted Storage System) system and plays the role of a storage miner.
Bucket follows the Apach2.0 open source specification and can be run by people from all over the world. It allows people to mine at a low hardware cost and together form a huge decentralized cloud storage system.
Bucket is completely implemented by Google's open source GO language and can run on most common Linux systems.

## Architecture diagram

![lQLPDhtvZTHL1t7NAvPNB0uwFRb_aF6-LVQChy7pDwD7AA_1867_755.png](https://cdn.nlark.com/yuque/0/2022/png/28698623/1652862144095-757600c6-d03f-4ba3-9145-0fd36fb98f6f.png#clientId=u730a622a-e3c2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=755&id=u3a1b76a7&margin=%5Bobject%20Object%5D&name=lQLPDhtvZTHL1t7NAvPNB0uwFRb_aF6-LVQChy7pDwD7AA_1867_755.png&originHeight=755&originWidth=1867&originalType=binary&ratio=1&rotation=0&showTitle=false&size=209597&status=done&style=none&taskId=u873948a7-3f33-4d06-9f22-57201dedb22&title=&width=1867)

## Project structure

```
cess-bucket
├── cmd
│   ├── cmd.go //command related code
│   └── main
│       └── main.go //entry
├── configs
│   ├── configfile.go
│   └── sys.go //global configuration
├── conf.toml //example of configuration file
├── go.mod
├── go.sum
├── initlz
│   └── system.go //system initialization
├── internal
│   ├── chain
│   │   ├── chainstate.go //implementation of query chain state
│   │   ├── eventstype.go //chain event definition
│   │   ├── main.go //define a global chain instance, reconnect if disconnected
│   │   ├── parameter.go //chain related data types
│   │   └── transaction.go //code of transaction
│   ├── encryption
│   │   └── rsa.go //RSA encryption algorithm
│   ├── logger
│   │   └── logger.go //log module
│   ├── proof
│   │   ├── apiv1
│   │   │   ├── Keygen.go //generate key using PBC library
│   │   │   ├── param.go //definition of parameter type
│   │   │   ├── PoDR2_challenge.go //generate proof challenges
│   │   │   ├── PoDR2_commit.go //calculate file tags
│   │   │   ├── PoDR2_prove.go //generate proof
│   │   │   └── PoDR2_verify.go //verify
│   │   └── main.go
│   ├── pt
│   │   └── types.go //public type
│   └── rpc
│       ├── client.go //rpc client
│       ├── client_test.go //tests
│       ├── codec.go //encode and decode
│       ├── conn.go //example of RPC connection
│       ├── errors.go //RPC error handling
│       ├── handler.go //RPC handlers
│       ├── main.go //upload and download
│       ├── proto
│       │   ├── msg.pb.go //Go file generated based on msg.proto
│       │   └── msg.proto //protobuf data transfer type definition
│       ├── server.go //RPC server instance
│       ├── service.go //RPC server processing
│       └── websocket.go
├── LICENSE
├── README.md
└── tools
├── ss58.go //ss58 address codec module
    └── util.go //Common tool code
```

## Command introduction

| **Command** | **Description**                                |
| ----------- | ---------------------------------------------- |
| version     | Print version number                           |
| default     | Generate configuration file template           |
| register    | Register mining miner information to the chain |
| state       | Query mining miner information                 |
| run         | Start mining normally                          |
| exit        | Exit the mining platform                       |
| increase    | Increase the deposit of mining miner           |
| withdraw    | Redemption deposit of mining miner             |
| obtain      | Get the test coins used by the testnet         |

## Functional positioning

| **Features**          | **implementation**                                           |
| --------------------- | ------------------------------------------------------------ |
| Register on the chain | [https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L172-L191](https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L172-L191) |
| Start mining          | [https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L227-L264](https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L227-L264) |
| View status           | [https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L194-L224](https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L194-L224) |
| Increase deposit      | [https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L553-L573](https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L553-L573) |
| Redemption deposit    | [https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L593-L607](https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L593-L607) |
| Exit mining           | [https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L576-L590](https://github.com/CESSProject/cess-bucket/blob/main/cmd/cmd.go#L576-L590) |
| Space management      | [https://github.com/CESSProject/cess-bucket/blob/main/internal/proof/main.go#L57-L263](https://github.com/CESSProject/cess-bucket/blob/main/internal/proof/main.go#L57-L263) |
| Handling challenges   | [https://github.com/CESSProject/cess-bucket/blob/main/internal/proof/main.go#L266-L373](https://github.com/CESSProject/cess-bucket/blob/main/internal/proof/main.go#L266-L373) |

## Configuration file

Bucket accepts a configuration file to allow you to customize some of its runtime information, which can avoid the trouble of re-modifying the program and compiling. You can use '-c' to specify the configuration file location.Its configuration file information is as follows:

```
# The rpc address of the chain node<br/>
RpcAddr      = ""<br/>
# Path to the mounted disk where the data is saved<br/>
MountedPath  = ""<br/>
# Total space used to store files, the unit is GB<br/>
StorageSpace = 1000<br/>
# The IP address of the machine's public network used by the mining program<br/>
ServiceAddr  = ""<br/>
# Port number monitored by the mining program<br/>
ServicePort  = 15001<br/>
# The address of income account<br/>IncomeAcc    = ""<br/>
# phrase or seed of the signature account<br/>
SignaturePrk = ""
```

## RPC interface

Buckets are uniformly managed by the scheduling service.They communicate with each other through RPC, and the data is encapsulated in protobuf format. The RPC interfaces provided by Bucket are as follows:

| **interface name** | **description**                                         |
| ------------------ | ------------------------------------------------------- |
| writefile          | Receive files uploaded by scheduling service            |
| readfile           | The dispatch service reads the file content             |
| writefiletag       | Receive the file tag uploaded by the scheduling service |
| readfiletag        | The scheduling service reads the file tag               |

Detailed description of the interface：
**writefile**

- input

```go
type PutFileToBucket struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
    
    //number of file chunks
    BlockTotal uint32 `protobuf:"varint,1,opt,name=blockTotal,proto3" json:"blockTotal,omitempty"`
    //index of file chunks
    BlockIndex uint32 `protobuf:"varint,2,opt,name=blockIndex,proto3" json:"blockIndex,omitempty"`
    //file chunk size
    BlockSize  uint32 `protobuf:"varint,3,opt,name=blockSize,proto3" json:"blockSize,omitempty"`
    //file id
    FileId     string `protobuf:"bytes,4,opt,name=fileId,proto3" json:"fileId,omitempty"`
    //file hash
    FileHash   string `protobuf:"bytes,5,opt,name=fileHash,proto3" json:"fileHash,omitempty"`
    //contents of file chunks
    BlockData  []byte `protobuf:"bytes,6,opt,name=blockData,proto3" json:"blockData,omitempty"`
}
```

- output

```go
type RespBody struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

    //return status code
	Code int32  `protobuf:"varint,1,opt,name=code,proto3" json:"code,omitempty"`
    //Reason statement
	Msg  string `protobuf:"bytes,2,opt,name=msg,proto3" json:"msg,omitempty"`
    //data(always is nil)
	Data []byte `protobuf:"bytes,3,opt,name=data,proto3" json:"data,omitempty"`
}
```

**readfile**

- input

```go
type FileDownloadReq struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

    //file id
	FileId        string `protobuf:"bytes,1,opt,name=file_id,json=fileId,proto3" json:"file_id,omitempty"`
    //Account address
	WalletAddress string `protobuf:"bytes,2,opt,name=walletAddress,proto3" json:"walletAddress,omitempty"`
    //index of file chunks
	Blocks        int32  `protobuf:"varint,3,opt,name=blocks,proto3" json:"blocks,omitempty"`
}
```

- output

```go
type FileDownloadInfo struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

    //file id
	FileId    string `protobuf:"bytes,1,opt,name=file_id,json=fileId,proto3" json:"file_id,omitempty"`
    //index of file chunks
	Blocks    int32  `protobuf:"varint,2,opt,name=blocks,proto3" json:"blocks,omitempty"`
    //file chunk size
	BlockSize int32  `protobuf:"varint,3,opt,name=block_size,json=blockSize,proto3" json:"block_size,omitempty"`
    //number of file chunks
	BlockNum  int32  `protobuf:"varint,4,opt,name=block_num,json=blockNum,proto3" json:"block_num,omitempty"`
    //contents of file chunks
	Data      []byte `protobuf:"bytes,5,opt,name=data,proto3" json:"data,omitempty"`
}
```

**writefiletag**

- intput

```go
type PutTagToBucket struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

    //file id
	FileId    string   `protobuf:"bytes,1,opt,name=fileId,proto3" json:"fileId,omitempty"`
    //The random number corresponding to the file name
	Name      []byte   `protobuf:"bytes,2,opt,name=name,proto3" json:"name,omitempty"`
    //number of file chunks
	N         int64    `protobuf:"varint,3,opt,name=n,proto3" json:"n,omitempty"`
    //Random number when scanning file chunks
	U         [][]byte `protobuf:"bytes,4,rep,name=u,proto3" json:"u,omitempty"`
    //file label signature
	Signature []byte   `protobuf:"bytes,5,opt,name=signature,proto3" json:"signature,omitempty"`
    //Authenticator
	Sigmas    [][]byte `protobuf:"bytes,6,rep,name=sigmas,proto3" json:"sigmas,omitempty"`
}
```

- output

```go
type RespBody struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

    //return status code
	Code int32  `protobuf:"varint,1,opt,name=code,proto3" json:"code,omitempty"`
    //Reason statement
	Msg  string `protobuf:"bytes,2,opt,name=msg,proto3" json:"msg,omitempty"`
    //data(always is nil)
	Data []byte `protobuf:"bytes,3,opt,name=data,proto3" json:"data,omitempty"`
}
```

**readfiletag**

- input

```go
type ReadTagReq struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

    //Account address
	Acc    string `protobuf:"bytes,1,opt,name=acc,proto3" json:"acc,omitempty"`
    //file id
	FileId string `protobuf:"bytes,2,opt,name=fileId,proto3" json:"fileId,omitempty"`
}
```

- output

```go
type RespBody struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

    //return status code
	Code int32  `protobuf:"varint,1,opt,name=code,proto3" json:"code,omitempty"`
    //Reason statement
	Msg  string `protobuf:"bytes,2,opt,name=msg,proto3" json:"msg,omitempty"`
    //data(always is nil)
	Data []byte `protobuf:"bytes,3,opt,name=data,proto3" json:"data,omitempty"`
}
```

## Space management

The goal of the space management task is to reasonably allocate the miner's hard disk space and automatically help the miner to complete the space challenge.The flow of space management tasks is as follows：
![EBC6F3B3-0106-4e0c-9808-63986F357E1B.png](https://cdn.nlark.com/yuque/0/2022/png/12491446/1652872186978-f4971edf-652f-42ae-94e3-58906215e7a9.png#clientId=u0520bc81-0419-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u0ced2f76&margin=%5Bobject%20Object%5D&name=EBC6F3B3-0106-4e0c-9808-63986F357E1B.png&originHeight=948&originWidth=667&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28139&status=done&style=none&taskId=ucd97c9fc-fa06-426b-9698-359db8ff392&title=)

## Handling challenges

The task of processing the challenge generates the corresponding proof data according to the random challenge generated on the chain, so as to prove that the challenge file is actually stored. If the challenge fails, the chain will punish the miner.

 
