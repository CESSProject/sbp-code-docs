# Code Documentation of CESS Portal

## Introduction

cess-portal is the client of the cess project. By using some simple commands of cess-portal, you can easily realize a series of operations such as purchasing space, querying space, uploading/downloading files, and querying file information on the Linux system.

## Development Introduction

- Framework: cobra
- development environment: Go1.17.2

## Code Example

### File Upload

- Upload the file to the CESS system, you can specify the number of backups and the private key. When uploading files, use websocket to communicate with scheduler, and use protobuf as the transmission structure
- Code Lnk:[https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/file.go#L31](https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/file.go#L31)
- Code Analyze:

```go
func FileUpload(path, backups, PrivateKey string) error {
    /*
    Process input data...
    */
    
    //Upload file meta information
    AsInBlock, err := ci.UploadFileMetaInformation(fileid, file.Name(), filehash, PrivateKey == "", uint8(spares), uint64(file.Size()), fee)
    if err != nil {
        fmt.Printf("\n[Error]Upload file meta information error:%s\n", err)
        return err
    }
    fmt.Printf("\nFile meta info upload:%s ,fileid is:%s\n", AsInBlock, fileid)
    
    //Query available schedules on the chain
    var client *rpc.Client
    for i, schd := range schds {
        wsURL := "ws://" + string(base58.Decode(string(schd.Ip)))
        ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        client, err = rpc.DialWebsocket(ctx, wsURL, "")
        defer cancel()
        if err != nil {
            ....
            if i == len(schds)-1 {
                //All schedules fail
                ...
                return err
            }
            continue
        } else {
            break
        }
    }
    sp := sync.Pool{
        New: func() interface{} {
            return &rpc.ReqMsg{}
        },
    }
    commit := func(num int, data []byte) error {
        /*
        Request scheduler, send files
        */
    }
    
    if len(PrivateKey) != 0 {
        //Encrypt file contents
        encodefile, err := tools.AesEncrypt(filebyte, []byte(PrivateKey))
        if err != nil {
            ...
        }
        //Send the encrypted file in chunks to scheduler
        for i := 0; i < blocktotal; i++ {
            block := make([]byte, 0)
            if blocks != i {
                block = encodefile[i*blocksize : (i+1)*blocksize]
                bar.Play(int64(i + 1))
            } else {
                block = encodefile[i*blocksize:]
                bar.Play(int64(i + 1))
            }
            //Send file to scheduler
            err = commit(i, block)
            if err != nil {
                ...
            }
        }
        bar.Finish()
    } else {
        //Send unencrypted file to scheduler
    }
    return nil
}
```

### File Download

- Download the file from the CESS system to the local, if the file is encrypted, you can choose to decrypt it. Use websocket to communicate with scheduler during file download, and use protobuf as the transmission structure
- Code Lnk:[https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/file.go#L271](https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/file.go#L271)
- Code Analyze:

```go
func FileDownload(fileid string) error {
    /*
    Process input data...
    */
    
    //Query available schedules on the chain
    schds, err := ci.GetSchedulerInfo()
    if err != nil {
        fmt.Printf("%s[Error]Get scheduler list error:%s%s\n ", tools.Red, err, tools.Reset)
        return err
    }
    
    var client *rpc.Client
    for i, schd := range schds {
        wsURL := "ws://" + string(base58.Decode(string(schd.Ip)))
        ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
        client, err = rpc.DialWebsocket(ctx, wsURL, "")
        defer cancel()
        if err != nil {
            ......
            continue
        } else {
            break
        }
    }
    
    //The files that need to be downloaded are obtained from the schedule in chunks
    for {
        data, err := proto.Marshal(&wantfile)
        
        ....
        
        resp, err := client.Call(ctx, req)
        cancel()
        if err != nil {
            ...
        }
        
        var respbody rpc.RespBody
        err = proto.Unmarshal(resp.Body, &respbody)
        if err != nil || respbody.Code != 200 {
            ....
        }
        var blockData module.FileDownloadInfo
        err = proto.Unmarshal(respbody.Data, &blockData)
        if err != nil {
            ....
        }
        //Write to local file chunk by chunk
        _, err = installfile.Write(blockData.Data)
        if err != nil {
            ....
            return err
        }
        
        getAllBar.Do(func() {
            bar.NewOption(0, int64(blockData.BlockNum))
        })
        bar.Play(int64(blockData.Blocks))
        wantfile.Blocks++
        sp.Put(req)
        //Stop if it is the last block
        if blockData.Blocks == blockData.BlockNum {
            break
        }
    }
    
    ......
    
    //If the file is encrypted, you can enter the password to decrypt
    if !fileinfo.Public {
        decodefile, err := tools.AesDecrypt(encodefile, filePWD)
        if err != nil {
            fmt.Println("[Error]Dncode the file fail ,error! ", err)
            return err
        }
        ....
    }
    
    return nil
}
```

### Purchase Storage

- You can buy storage using TCESS in your account, 'quantity' means how many GB of space to purchase, 'duration' represents the purchase period in months, 'expected' means users accept the highest price you can afford per GB per month of storage space, if the price of the space exceeds your expectations at the time of purchase, the purchase of space will fail.
- Code Lnk:[https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/purchase.go#L65](https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/purchase.go#L65)
- Code Analyze:

```go
func Expansion(quantity, duration, expected int) error {
    chain.Chain_Init()
    
    var ci chain.CessInfo
    ci.RpcAddr = conf.ClientConf.ChainData.CessRpcAddr
    ci.IdentifyAccountPhrase = conf.ClientConf.ChainData.IdAccountPhraseOrSeed
    ci.TransactionName = chain.BuySpaceTransactionName
    
    //Buying space on-chain, failure could mean do not have enough money or there is not enough space on the chain
    err := ci.BuySpaceOnChain(quantity, duration, expected/1024)
    if err != nil {
        ...
    }
    fmt.Printf("[Success]Buy space on chain success!\n")
    return nil
}
```

### Query Price

- Quoting the price per GB of space
- Code Lnk:[https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/query.go#L36](https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/query.go#L36)
- Code Analyze:

```go
func QueryPrice() error {
    chain.Chain_Init()
    
    var ci chain.CessInfo
    ci.RpcAddr = conf.ClientConf.ChainData.CessRpcAddr
    ci.ChainModule = chain.FindPriceChainModule
    
    ci.ChainModuleMethod = chain.FindPriceModuleMethod
    //Get the price of space directly from an on-chain method
    Price, err := ci.GetPrice()
    if err != nil {
        fmt.Printf("%s[Error]%sGet price fail:%s\n", tools.Red, tools.Reset, err)
        logger.OutPutLogger.Sugar().Infof("%s[Error]%sGet price fail::%s\n", tools.Red, tools.Reset, err)
        return err
    }
    //Show how much it costs per G
    PerGB, err := strconv.ParseFloat(strconv.FormatInt(Price.Int64()*1024, 10), 64)
    fmt.Printf("[Success]The current storage price is:%f TCESS per (G)\n", PerGB)
    logger.OutPutLogger.Sugar().Infof("[Success]The current storage price is:%f TCESS per (G)\n", PerGB)
    return nil
}
```

### Query File

- User can use this method to query the file information corresponding to the file ID. If the 'fileid' is empty, it means to query all the uploaded file information of the user.
- Code Lnk:[https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/query.go#L60](https://github.com/CESSProject/cess-portal/blob/4ea00d7056de7afde0e372201fd6bad5729e15bd/client/query.go#L60)
- Code Analyze:

```go
func QueryFile(fileid string) error {
    chain.Chain_Init()
    
    var ci chain.CessInfo
    ci.RpcAddr = conf.ClientConf.ChainData.CessRpcAddr
    ci.ChainModule = chain.FindFileChainModule
    
    //When the file id is passed in, the specific information of the file is queried
    if fileid != "" {
        ci.ChainModuleMethod = chain.FindFileModuleMethod[0]
        data, err := ci.GetFileInfo(fileid)
        if err != nil {
            ...
        }
        fmt.Println(data)
        //If the name of the file cannot be found, because the content of the file has been deleted, the meta information on the chain will be scheduled to be deleted at a certain time.
        if len(data.File_Name) == 0 {
            fmt.Printf("%s[Tips]This file:%s may have been deleted by someone%s\n", tools.Yellow, fileid, tools.Reset)
        }
    } else {
        //When no file id is passed in, query all uploaded files of the user
        ci.ChainModuleMethod = chain.FindFileModuleMethod[1]
        data, err := ci.GetFileList()
        if err != nil {
            ...
        }
        for _, fileinfo := range data {
            fmt.Printf("%s\n", string(fileinfo))
        }
    }
    return nil
}
```

