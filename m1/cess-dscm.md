# Decentralized Storage Cost Mechanism

### 1. Overall process

Send HTTP requests through the OCW (off-chain workers) to read the rivals' pricing, and then upload the read price to the blockchain through an account signature. After this, we divide the read price by three and obtain the pricing of CESS chain, take the lower quote and update the unit price.


### 2. Off-Chain Workers

The off-chain worker executes the read price method once a day, and will immediately execute when the blockchain block height is 1.

```rust
fn offchain_worker(block_number: T::BlockNumber) {
    let number: u128 = block_number.saturated_into();
    let one_day: u128 = <T as Config>::OneDay::get().saturated_into();
    //It will be executed once a day 
    //and immediately when the blockchain block height is 1
    if number % one_day == 0 || number == 1 {
        //Query price
        let result = Self::offchain_fetch_price(block_number);
        if let Err(e) = result {
            log::error!("offchain_worker error: {:?}", e);
        }
    }
}
```

### 3. HTTP Request

Send an HTTP request to [[https://arweave.net/price/1048576](https://arweave.net/price/1048576)] and request for the Arweave pricing. "1048576" represents the price for 1MB of storage space.

```rust
			log::info!("send request to {}", HTTP_REQUEST_STR);
			//Create request
			//The request address is: https://arweave.net/price/1048576
			//Arweave's official API is used to query prices
			//Request price of 1MB
			let request = rt_offchain::http::Request::get(HTTP_REQUEST_STR);
			//Set timeout wait
			let timeout = sp_io::offchain::timestamp()
			.add(rt_offchain::Duration::from_millis(FETCH_TIMEOUT_PERIOD));
			log::info!("send request");
			//Send request
			let pending = request
			.add_header("User-Agent","PostmanRuntime/7.28.4")
				.deadline(timeout) // Setting the timeout time
				.send() // Sending the request out by the host
				.map_err(|_| <Error<T>>::HttpFetchingError)?;
			
			log::info!("wating response");
			//Waiting for response
			let response = pending
				.wait()
				.map_err(|_| <Error<T>>::HttpFetchingError)?;
			//Determine whether the response is wrong
			if response.code != 200 {
				log::error!("Unexpected http request status code: {}", response.code);
				Err(<Error<T>>::HttpFetchingError)?;
			}

```

### 4. Send a signed transaction

Put the price retrieved on the chain.

```rust
            //Send signed transaction
			//call update_price transaction
			let result = signer.send_signed_transaction(|_acct| 
				Call::update_price{price: value.clone()}
			);
```

### 5. Storage pricing

After verifying the signature, convert the price of type Vec<u8> to type u128. Then obtain CESS pricing calculated according to the current space on the chain, store the lower price as the unit price of our current storage space on the chain.

```rust
            let _sender = ensure_signed(origin)?;
			//Convert price of string type to balance
			//Vec<u8> -> str
			let str_price = str::from_utf8(&price).unwrap();
			//str -> u128
			let mut price_u128: u128 = str_price
				.parse()
				.map_err(|_e| Error::<T>::ConversionError)?;
			
			//One third of the price
			price_u128 = price_u128.checked_div(3).ok_or(Error::<T>::Overflow)?;
				
			//Get the current price on the chain
			let our_price = Self::get_price();
			//Which pricing is cheaper
			if our_price < price_u128 {
				price_u128 = our_price;
			}
			//u128 -> balance
			let price_balance: BalanceOf<T> = price_u128.try_into().map_err(|_e| Error::<T>::ConversionError)?;
			<UnitPrice<T>>::put(price_balance);
```

### 6. The calculation method of the current chain space and price

The ratio of 1024 MB to the free space on the chain is how much TCESS it is worth per MB, and the final confirmed price is the basic value multiplied by 1000. 

```rust
            //Get the available space on the current chain
			let space = pallet_sminer::Pallet::<T>::get_space();
			//If it is not 0, the logic is executed normally
			if space != 0 {
				//Calculation rules
				//The price is based on 1024 / available space on the current chain
				//Multiply by the base value 1 tcess * 1_000 (1_000_000_000_000 * 1_000)
				let price: u128 = M_BYTE * 1024 * 1_000_000_000_000 * 1000 / space ;
				return price;
			}
```

### 7. How to test

git clone [https://github.com/CESSProject/cess.git](https://github.com/CESSProject/cess.git)
Execute the cargo test command in the cess root directory

 ` cargo test `

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22121693/1652872157717-d26f6b90-4c0f-4166-86fb-78e0cd926b0d.png#clientId=uaba2ddf5-fede-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=383&id=u778ea806&margin=%5Bobject%20Object%5D&name=image.png&originHeight=383&originWidth=833&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60879&status=done&style=none&taskId=ub0e6ed01-0465-4973-ada2-420959de595&title=&width=833)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22121693/1652872179572-94bb1b1e-d430-4a3c-8c28-3bec66205da0.png#clientId=uaba2ddf5-fede-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=168&id=ysRjH&margin=%5Bobject%20Object%5D&name=image.png&originHeight=168&originWidth=818&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22763&status=done&style=none&taskId=u94fc27f9-1898-4a57-a014-78093152905&title=&width=818)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22121693/1652872208564-9e51743b-8ccc-44fb-bb47-ee3b4c9a4705.png#clientId=uaba2ddf5-fede-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=245&id=Or8xC&margin=%5Bobject%20Object%5D&name=image.png&originHeight=245&originWidth=836&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35390&status=done&style=none&taskId=u0a02784e-a4fb-4adc-8be6-9e240d8695b&title=&width=836)
