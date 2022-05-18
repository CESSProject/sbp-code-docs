# Code Ducumentation of CESS Explorer

## Introduction

Polkadot/Substrate UI for interacting with CESS node.

It is forked from polkadot/apps, and currently supports http and wss access methods.

## Add custom APIs

Since many new APIs have been developed on the CESS chain, which makes querying miners and storage space simpler and more convenient. Corresponding changes have also been made on cess-explorer, replacing the blockchain browser API with a general API, which includes our upgraded miner query API, storage space related API, file operation related API, etc. The detailed replacement includes the following points:

1. **Chain state API of related storage miner**：

- _api.query.sminer.minerItems_

2. **Chain state API of related storage space**：

- _api.query.sminer.totalspace_

- _api.query.sminer.totalpower_

3. **Chain state API of related storage files**：

- _api.query.sminer.minerDetails_

- _api.query.fileBank.file_

- _api.query.fileBank.userHoldSpaceDetails_

4. **Chain state API of related miner list**

- _api.query.sminer.minerItems_

- _api.query.sminer.minerDetails_

```javascript
useEffect(() => {
  (async (): Promise<void> => {
   const res = await api.query.sminer.minerItems.entries();// all miners      
  if (res) {
    let info = {
      activeMiners: res.length
    };
    setMinerData(info);
  }
})().catch(console.error);
}, [])

useEffect(() => {
  (async (): Promise<void> => {
   const res = await api.query.staking.currentEra();
  if (res) {
    let info: any = res.toHuman();// or toJSON()
    setCurrentEra(info);
    let res2 = await api.query.staking.erasRewardPoints(info - 1);
    setLastEra(_.get(res2.toHuman(), 'total'));
  }
})().catch(console.error);
  }, [])
  
  function BestHash ({ className = '', label }: Props): React.ReactElement<Props> {
  const { api } = useApi();
  const newHead = useCall<Header>(api.rpc.chain.subscribeNewHeads);

  return (
    <div className={className}>
      {label || ''}{newHead?.hash.toHex()}
    </div>
  );
}

    (async (): Promise<void> => {
      _subFinHead = await api.rpc.chain.subscribeFinalizedHeads(_newFinalized);
      _subNewHead = await api.rpc.chain.subscribeNewHeads(_newHeader);
    })().catch(console.error);

let res: any = await api.query.sminer.minerDetails(value);// get miner info
      let resJson: any = res.toJSON();
      resJson.totalRewardObj = formatterCurrency(resJson.totalReward);
      resJson.totalRewardsCurrentlyAvailableObj = formatterCurrency(resJson.totalRewardsCurrentlyAvailable);
      resJson.totaldNotReceiveObj = formatterCurrency(resJson.totaldNotReceive);
      resJson.collateralsObj = formatterCurrency(resJson.collaterals);

const minerItems:any = await api.query.sminer.minerItems.entries();//get all miner list
      const minerDetails:any = await api.query.sminer.minerDetails.entries();
      const files:any=await api.query.fileBank.file.entries();

let entries:any = await api.query.fileBank.file.entries();
      let list:any[]= [];
      entries.forEach(([key, entry]) => {
        let fileid:string = key.args.map((k) => k.toHuman());
        let jsonObj = entry.toJSON();
        let humanObj = entry.toHuman();
        if(jsonObj.owner == value){
          jsonObj.filesize = formatterSize(jsonObj.filesize);
          jsonObj.filename = humanObj.filename;
          jsonObj.similarityhash = humanObj.similarityhash;
          list.push(_.assign(jsonObj,{fileid}));
        }
      });

const minerItems:any = await api.query.sminer.minerItems.entries();//get all miner list
      const minerDetails:any = await api.query.sminer.minerDetails.entries();
      const files:any=await api.query.fileBank.file.entries();

```

## Adjust network switching interaction logic

We changed the node switching logic to separate the function of node saving from node switching. The user will be noticed that the saving is successful, then the node can be switched, instead of directly switching the node after saving as before, when the user doesn't know whether the saving is successful or not. 

```javascript
function isValidUrl (url: string): boolean {
  return (
    // some random length... we probably want to parse via some lib
    (url.length >= 7) &&
    // check that it starts with a valid ws identifier
    (url.startsWith('ws://') || url.startsWith('wss://') || url.startsWith('light://'))
  );
}

function combineEndpoints (endpoints: LinkOption[]): Group[] {
  return endpoints.reduce((result: Group[], e): Group[] => {
    if (e.isHeader) {
      result.push({ header: e.text, isDevelopment: e.isDevelopment, isSpaced: e.isSpaced, networks: [] });
    } else {
      const prev = result[result.length - 1];
      const prov = { isLightClient: e.isLightClient, name: e.textBy, url: e.value };

      if (prev.networks[prev.networks.length - 1] && e.text === prev.networks[prev.networks.length - 1].name) {
        prev.networks[prev.networks.length - 1].providers.push(prov);
      } else {
        prev.networks.push({
          icon: e.info,
          isChild: e.isChild,
          isUnreachable: e.isUnreachable,
          name: e.text as string,
          providers: [prov]
        });
      }
    }

    return result;
  }, []);
}

function getCustomEndpoints (): string[] {
  try {
    const storedAsset = localStorage.getItem(CUSTOM_ENDPOINT_KEY);

    if (storedAsset) {
      return JSON.parse(storedAsset) as string[];
    }
  } catch (e) {
    console.error(e);
    // ignore error
  }

  return [];
}

function extractUrlState (apiUrl: string, groups: Group[]): UrlState {
  let groupIndex = groups.findIndex(({ networks }) =>
    networks.some(({ providers }) =>
      providers.some(({ url }) => url === apiUrl)
    )
  );

  if (groupIndex === -1) {
    groupIndex = groups.findIndex(({ isDevelopment }) => isDevelopment);
  }

  return {
    apiUrl,
    groupIndex,
    hasUrlChanged: settings.get().apiUrl !== apiUrl,
    isUrlValid: isValidUrl(apiUrl)
  };
}

function loadAffinities (groups: Group[]): Record<string, string> {
  return Object
    .entries<string>(store.get(STORAGE_AFFINITIES) as Record<string, string> || {})
    .filter(([network, apiUrl]) =>
      groups.some(({ networks }) =>
        networks.some(({ name, providers }) =>
          name === network && providers.some(({ url }) => url === apiUrl)
        )
      )
    )
    .reduce((result: Record<string, string>, [network, apiUrl]): Record<string, string> => ({
      ...result,
      [network]: apiUrl
    }), {});
}

function isSwitchDisabled (hasUrlChanged: boolean, apiUrl: string, isUrlValid: boolean): boolean {
  if (!hasUrlChanged) {
    return true;
  } else if (apiUrl.startsWith('light://')) {
    return false;
  } else if (isUrlValid) {
    return false;
  }

  return true;
}
```

## Updating the search functionality

We modified the search function to search block/address/extrinsic (on-chain transactions) based on the entered keywords.

```javascript
const _onQuery = useCallback(
    async (): Promise<void> => {
      if (value.length === 0) {
        return;
      }
      let url='';
      switch(value.length){
        case 66://hash length is 66
          url=`/explorer/query/${value}/${'block'}`;
          break;
        case 48://wallet address length is 48
          url=`/explorer/query/${value}/${'address'}`;
          break;
        default://other is Extrinsics Hash
          url=`/explorer/query/${value}/${'extrinsic'}`;          
          break;
      }
      if(url){
        // window.location.hash =url;
        // use react-router-dom
        history.push(url);
      }
    },
    [value]
  );
```

## Added and optimized query functionality

For example, a miner search function has been added to the miner list page, and the miner details can be viewed according to the miner ID.

```javascript
import React, { useState, useCallback } from "react"
import styled from "styled-components";
import Icon from "@polkadot/react-components/Icon";

interface Props {
  className?: string
}

function MinerSearch({ className }: Props): React.ReactElement<Props> {
  const [keyword, setKeyword] = useState('');

  const _setNumber = useCallback(
    (event: any): void => setKeyword(event.target.value),//miner iD
    []
  );
  const _onQuery = useCallback(
    () => {
      let value = parseInt(keyword.trim());
      if (value === 0) {
        return;
      }
      window.location.hash = `/explorer/query/${value}/miner`;
    },
    [keyword]
  );
  return (
    <span className={className}>
      <input type="number" placeholder="Search by miner ID" onKeyPress={_setNumber} onKeyUp={_setNumber} onChange={_setNumber} onBlur={_setNumber} />
      <button onClick={_onQuery}><Icon className='highlight--color' icon='search' /></button>
    </span>
  )
}

export default React.memo(styled(MinerSearch)`
    float:right;
    position: relative;
    width: 213px;
    >input{
      width: 160px;
      border: 1px solid #ddd;
      border-radius: 4px;
      height: 30px;
      line-height: 30px;
      font-size: 14px;
      position: absolute;
      left: 0px;
      top: 0px;
    }
    >button{
      width: 50px;
      height: 30px;
      line-height: 30px;
      margin-left: 5px;
      border: none;
      font-size: 17px;
      border-radius: 4px;
      position: absolute;
      top: 0px;
      right: 0px;
    }
  }
`)

```
