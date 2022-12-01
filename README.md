
# How-to-install-04-zkapp-browser-ui-on-MINA-Protocol

<div align='center'>

<a href='https://minaprotocol.com/'>
    <img width="700" height="350" src="https://img.capital.com/imgs/articles/662x308x1/shutterstock_2017166612_0.jpg"/>
</a>

Simple way how to install How to install 04-zkapp-browser-ui on MINA Protocol


# Reference


[Official Docs](https://docs.minaprotocol.com/zkapps/tutorials/zkapp-ui-with-react)

[Discord ](https://discord.gg/zJkqXjm8)

[Wallet](https://www.aurowallet.com/)

[Faucet](https://faucet.minaprotocol.com/)

</div>

## Requirements

| Tool | Operation |
|----------|---------------------|
|[VPS](https://contabo.com/)-<b>M Size Recomemended</b>|Ubuntu 16.04 or higher|
|[Mobaxterm](https://mobaxterm.mobatek.net/download.html)|Terminal|

## First Step

1. Install and Create [wallet](https://www.aurowallet.com/), change to <i>Berkeley</i> Network
2. Copy address and Request [Faucet](https://faucet.minaprotocol.com/) - <b>Note:</b> 1 address only can request once

## Second Step
1. Open your Terminal that connect to your VPS
2. Prepare some <b><i>Snack or Coffee</b></i> to enjoy your Code


# <p align="center">Lets Start</p>

## 1. Open Port

```
apt update
apt upgrade -y
```
```
sudo ufw enable
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 22/tcp
sudo ufw allow 443
sudo ufw allow 22
sudo ufw allow 300
```
## 2. Install System Operation
```
sudo apt install git
sudo apt install -y curl
```
```
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
```
```
sudo apt install -y nodejs
```

## 3 . Install Zkapp CLI


```
git clone https://github.com/o1-labs/zkapp-cli
```
```
npm instal -g zkapp-cli@0.5.3
```

Check Version Zk 
```
zk --version
```
## 4 . Create Project

```
zk project 04-zkapp-browser-ui --ui next
```
--<i>after you "Create Project" there are 3 times options will show up. Pick <b>Yes</b> for all of them or you can press `enter`</i>

## 5. Create Your Own Repository on [Github](https://github.com)
Create a Repository on your github account, named it as `04-zkapp-browser-ui` .

<i>after you create it, some code will be showing up. Some code will be usefull for running on your terminal or you can give command like this :</i>

```
cd 04-zkapp-browser-ui
```
```
git remote add origin <your-repo-url>
```
change `<your-repo-url>` with your own repository url. <i>Ex : `git remote add origin https://github.com/Leevaleeth/04-zkapp-browser-ui.git`</i>

Next,
```
git push -u origin main
```
after you give this Command, you need to enter your <b>Github Username & Password</b> <i>Chill bro....</i> it wont <b>Hack</b> your github account. Follow this step to get OTP password only for your `Repository`

1. Go to your `Setting` on your Github Account
2. Open `<> Depelover Settings`
3. `Personal Acces Token` > Choose on `Token (Classic)`
4. `Generate`<b> and dont forget to Checklist on</b> `Repo`
5. Done. Your token access is ready to use. Use it as a Password on your terminal.

<b>NOTE: <i>Your Personal Access Token will only showing Once, make sure you Copy and save it on another place/note</i></b>

## 6 . Run Contract

```
cd
cd 04-zkapp-browser-ui/contracts/
```
```
npm run build
```

## 7. Create Files

```
cd
cd 04-zkapp-browser-ui/ui/pages
```
```
nano zkappWorker.ts
```
An blank window will pop-ing up.

Copy and Paste this command inside the blank window :

```
import {
    Mina,
    isReady,
    PublicKey,
    PrivateKey,
    Field,
    fetchAccount,
} from 'snarkyjs'

type Transaction = Awaited<ReturnType<typeof Mina.transaction>>;

// ---------------------------------------------------------------------------------------

import type { Add } from '../../contracts/src/Add';

const state = {
    Add: null as null | typeof Add,
    zkapp: null as null | Add,
    transaction: null as null | Transaction,
}

// ---------------------------------------------------------------------------------------

const functions = {
    loadSnarkyJS: async (args: {}) => {
        await isReady;
    },
    setActiveInstanceToBerkeley: async (args: {}) => {
        const Berkeley = Mina.BerkeleyQANet(
            "https://proxy.berkeley.minaexplorer.com/graphql"
        );
        Mina.setActiveInstance(Berkeley);
    },
    loadContract: async (args: {}) => {
        const { Add } = await import('../../contracts/build/src/Add.js');
        state.Add = Add;
    },
    compileContract: async (args: {}) => {
        await state.Add!.compile();
    },
    fetchAccount: async (args: { publicKey58: string }) => {
        const publicKey = PublicKey.fromBase58(args.publicKey58);
        return await fetchAccount({ publicKey });
    },
    initZkappInstance: async (args: { publicKey58: string }) => {
        const publicKey = PublicKey.fromBase58(args.publicKey58);
        state.zkapp = new state.Add!(publicKey);
    },
    getNum: async (args: {}) => {
        const currentNum = await state.zkapp!.num.get();
        return JSON.stringify(currentNum.toJSON());
    },
    createUpdateTransaction: async (args: {}) => {
        const transaction = await Mina.transaction(() => {
            state.zkapp!.update();
        }
        );
        state.transaction = transaction;
    },
    proveUpdateTransaction: async (args: {}) => {
        await state.transaction!.prove();
    },
    getTransactionJSON: async (args: {}) => {
        return state.transaction!.toJSON();
    },
};

// ---------------------------------------------------------------------------------------

export type WorkerFunctions = keyof typeof functions;

export type ZkappWorkerRequest = {
    id: number,
    fn: WorkerFunctions,
    args: any
}

export type ZkappWorkerReponse = {
    id: number,
    data: any
}
if (process.browser) {
    addEventListener('message', async (event: MessageEvent<ZkappWorkerRequest>) => {
        const returnData = await functions[event.data.fn](event.data.args);

        const message: ZkappWorkerReponse = {
            id: event.data.id,
            data: returnData,
        }
        postMessage(message)
    });
}
```
then Save it. <i>How?</i> `CTRL` `X` `Y` then `ENTER`
#### Next
```
nano zkappWorkerClient.ts
```

An blank window will pop-ing up.

Copy and Paste this command inside the blank window :
```
import {
    fetchAccount,
    PublicKey,
    PrivateKey,
    Field,
} from 'snarkyjs'

import type { ZkappWorkerRequest, ZkappWorkerReponse, WorkerFunctions } from './zkappWorker';

export default class ZkappWorkerClient {

    // ---------------------------------------------------------------------------------------

    loadSnarkyJS() {
        return this._call('loadSnarkyJS', {});
    }

    setActiveInstanceToBerkeley() {
        return this._call('setActiveInstanceToBerkeley', {});
    }

    loadContract() {
        return this._call('loadContract', {});
    }

    compileContract() {
        return this._call('compileContract', {});
    }

    fetchAccount({ publicKey }: { publicKey: PublicKey }): ReturnType<typeof fetchAccount> {
        const result = this._call('fetchAccount', { publicKey58: publicKey.toBase58() });
        return (result as ReturnType<typeof fetchAccount>);
    }

    initZkappInstance(publicKey: PublicKey) {
        return this._call('initZkappInstance', { publicKey58: publicKey.toBase58() });
    }

    async getNum(): Promise<Field> {
        const result = await this._call('getNum', {});
        return Field.fromJSON(JSON.parse(result as string));
    }

    createUpdateTransaction() {
        return this._call('createUpdateTransaction', {});
    }

    proveUpdateTransaction() {
        return this._call('proveUpdateTransaction', {});
    }

    async getTransactionJSON() {
        const result = await this._call('getTransactionJSON', {});
        return result;
    }

    // ---------------------------------------------------------------------------------------

    worker: Worker;

    promises: { [id: number]: { resolve: (res: any) => void, reject: (err: any) => void } };

    nextId: number;

    constructor() {
        this.worker = new Worker(new URL('./zkappWorker.ts', import.meta.url))
        this.promises = {};
        this.nextId = 0;

        this.worker.onmessage = (event: MessageEvent<ZkappWorkerReponse>) => {
            this.promises[event.data.id].resolve(event.data.data);
            delete this.promises[event.data.id];
        };
    }

    _call(fn: WorkerFunctions, args: any) {
        return new Promise((resolve, reject) => {
            this.promises[this.nextId] = { resolve, reject }

            const message: ZkappWorkerRequest = {
                id: this.nextId,
                fn,
                args,
            };

            this.worker.postMessage(message);

            this.nextId++;
        });
    }
}
```
then Save it. <i>How?</i> `CTRL` `X` `Y` then `ENTER`

## 8. Add Css

```
cd
cd 04-zkapp-browser-ui/ui/styles
```
```
nano globals.css
```
Delete everything inside and then paste this inside
```
html,
body {
  padding: 0;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen,
    Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
}

a {
  color: inherit;
  text-decoration: none;
}

* {
  box-sizing: border-box;
}

@media (prefers-color-scheme: dark) {
  html {
    color-scheme: dark;
  }
  body {
    color: white;
    background: black;
  }
}
```
Save `CTRL` `X` `Y` and `Enter`

## 10 .  Running Web
1. Open 1 more tab terminal with same VPS
2. so there are 2 terminal open on 1 VPS
![image](https://user-images.githubusercontent.com/110814039/204132179-2bdeb97c-bbec-41b2-a023-5dcf729f1ec8.png)

##### 1st Tab Terminal
```
cd
cd 04-zkapp-browser-ui/ui/
npm run dev
```

##### 2nd Tab Terminal

```
cd
cd 04-zkapp-browser-ui/ui/
npm run ts-watch
```

#### Next
1. Open http://xxx.xx.xxx.xxx:3000 Google Chrome Browser. <b>Change</b> `xxx.xx.xxx.xxx` with your own VPS IP
2. Ignore `Error` that showing up.
<b>Note:</b> <i>This step is just to make sure that your progress is right and your app is connecting to the site/github</i>

## 11. Implementing the react app
#### Open 1 more tab terminal with same VPS

```
cd
cd 04-zkapp-browser-ui/ui/pages
nano _app.page.tsx
```
Delete everything inside and then paste this inside:

```
// import '../styles/globals.css'
// import type { AppProps } from 'next/app'

// import './reactCOIServiceWorker';

// export default function App({ Component, pageProps }: AppProps) {
//   return <Component {...pageProps} />
// }


import '../styles/globals.css'
import { useEffect, useState } from "react";
import './reactCOIServiceWorker';

import ZkappWorkerClient from './zkappWorkerClient';

import {
  PublicKey,
  PrivateKey,
  Field,
} from 'snarkyjs'

let transactionFee = 0.1;

export default function App() {

  let [state, setState] = useState({
    zkappWorkerClient: null as null | ZkappWorkerClient,
    hasWallet: null as null | boolean,
    hasBeenSetup: false,
    accountExists: false,
    currentNum: null as null | Field,
    publicKey: null as null | PublicKey,
    zkappPublicKey: null as null | PublicKey,
    creatingTransaction: false,
  });

  // -------------------------------------------------------
  // Do Setup

  useEffect(() => {
    (async () => {
      if (!state.hasBeenSetup) {
        const zkappWorkerClient = new ZkappWorkerClient();

        console.log('Loading SnarkyJS...');
        await zkappWorkerClient.loadSnarkyJS();
        console.log('done');

        await zkappWorkerClient.setActiveInstanceToBerkeley();

        const mina = (window as any).mina;

        if (mina == null) {
          setState({ ...state, hasWallet: false });
          return;
        }

        const publicKeyBase58: string = (await mina.requestAccounts())[0];
        const publicKey = PublicKey.fromBase58(publicKeyBase58);

        console.log('using key', publicKey.toBase58());

        console.log('checking if account exists...');
        const res = await zkappWorkerClient.fetchAccount({ publicKey: publicKey! });
        const accountExists = res.error == null;

        await zkappWorkerClient.loadContract();

        console.log('compiling zkApp');
        await zkappWorkerClient.compileContract();
        console.log('zkApp compiled');

        const zkappPublicKey = PublicKey.fromBase58('B62qrDe16LotjQhPRMwG12xZ8Yf5ES8ehNzZ25toJV28tE9FmeGq23A');

        await zkappWorkerClient.initZkappInstance(zkappPublicKey);

        console.log('getting zkApp state...');
        await zkappWorkerClient.fetchAccount({ publicKey: zkappPublicKey })
        const currentNum = await zkappWorkerClient.getNum();
        console.log('current state:', currentNum.toString());

        setState({
          ...state,
          zkappWorkerClient,
          hasWallet: true,
          hasBeenSetup: true,
          publicKey,
          zkappPublicKey,
          accountExists,
          currentNum
        });
      }
    })();
  }, []);

  // -------------------------------------------------------
  // Wait for account to exist, if it didn't

  useEffect(() => {
    (async () => {
      if (state.hasBeenSetup && !state.accountExists) {
        for (; ;) {
          console.log('checking if account exists...');
          const res = await state.zkappWorkerClient!.fetchAccount({ publicKey: state.publicKey! })
          const accountExists = res.error == null;
          if (accountExists) {
            break;
          }
          await new Promise((resolve) => setTimeout(resolve, 5000));
        }
        setState({ ...state, accountExists: true });
      }
    })();
  }, [state.hasBeenSetup]);

  // -------------------------------------------------------
  // Send a transaction

  const onSendTransaction = async () => {
    setState({ ...state, creatingTransaction: true });
    console.log('sending a transaction...');

    await state.zkappWorkerClient!.fetchAccount({ publicKey: state.publicKey! });

    await state.zkappWorkerClient!.createUpdateTransaction();

    console.log('creating proof...');
    await state.zkappWorkerClient!.proveUpdateTransaction();

    console.log('getting Transaction JSON...');
    const transactionJSON = await state.zkappWorkerClient!.getTransactionJSON()

    console.log('requesting send transaction...');
    const { hash } = await (window as any).mina.sendTransaction({
      transaction: transactionJSON,
      feePayer: {
        fee: transactionFee,
        memo: '',
      },
    });

    console.log(
      'See transaction at https://berkeley.minaexplorer.com/transaction/' + hash
    );

    setState({ ...state, creatingTransaction: false });
  }

  // -------------------------------------------------------
  // Refresh the current state

  const onRefreshCurrentNum = async () => {
    console.log('getting zkApp state...');
    await state.zkappWorkerClient!.fetchAccount({ publicKey: state.zkappPublicKey! })
    const currentNum = await state.zkappWorkerClient!.getNum();
    console.log('current state:', currentNum.toString());

    setState({ ...state, currentNum });
  }

  // -------------------------------------------------------
  // Create UI elements

  let hasWallet;
  if (state.hasWallet != null && !state.hasWallet) {
    const auroLink = 'https://www.aurowallet.com/';
    const auroLinkElem = <a href={auroLink} target="_blank" rel="noreferrer"> [Link] </a>
    hasWallet = <div> Could not find a wallet. Install Auro wallet here: {auroLinkElem}</div>
  }

  let setupText = state.hasBeenSetup ? 'SnarkyJS Ready' : 'Setting up SnarkyJS...';
  let setup = <div> {setupText} {hasWallet}</div>

  let accountDoesNotExist;
  if (state.hasBeenSetup && !state.accountExists) {
    const faucetLink = "https://faucet.minaprotocol.com/?address=" + state.publicKey!.toBase58();
    accountDoesNotExist = <div>
      Account does not exist. Please visit the faucet to fund this account
      <a href={faucetLink} target="_blank" rel="noreferrer"> [Link] </a>
    </div>
  }

  let mainContent;
  if (state.hasBeenSetup && state.accountExists) {
    mainContent = <div>
      <button onClick={onSendTransaction} disabled={state.creatingTransaction}> Send Transaction </button>
      <div> Current Number in zkApp: {state.currentNum!.toString()} </div>
      <button onClick={onRefreshCurrentNum}> Get Latest State </button>
    </div>
  }

  return <div>
    {setup}
    {accountDoesNotExist}
    {mainContent}
  </div>
}
```
Save `CTRL` `X` `Y` and `Enter`

## 12 . Deploy UI to Repository

```
cd 04-zkapp-browser-ui/ui/
npm run deploy
```

wait until the proccess finished.

![image_2022-11-26_16-00-59](https://user-images.githubusercontent.com/110814039/204132988-86558aa3-8d26-4dfb-b2d9-da75d1d759f3.png)

if its all done, you will need to enter yout <b>Github Username and Password</b> . Use Personal token access like before as a password.

## 13. Running the site
1. Open this site on Google Chrome: https://xxxxxx.github.io/04-zkapp-browser-ui/index.html <b>Change</b> `xxx` with your GIthub username. EX: `https://Leevaleeth.github.io/04-zkapp-browser-ui/index.html`
2. Wait until your [Wallet](https://www.aurowallet.com/) poping up to connecting request. Aprrove the request, so your [Wallet](https://www.aurowallet.com/) and your site will be connected
3. After successfully connected, you need to wait more time until `Send Transaction` Button showing up
![image](https://user-images.githubusercontent.com/110814039/204133411-2c1faf95-4777-423b-a779-e5f8ac486529.png)
4. Is that need a long time to wait ?
5. <b>YES!</b> thats why i told you to prepare some Snack or Coffee :)
6. <i>Waiting~~~</i>
7. <i>Waiting~~~</i>
8. If `Send Transaction` Button have showing up, click it and do some transactions
9. No respond from the wallet ? <b>YES</b> you need to <b>WAIT</b> more time until your wallet pop-ing up to accept the request.

<b>NOTE:</b> <i>you need to make sure your transaction is success on [Explorer](https://berkeley.minaexplorer.com/) if it failed you need to repeat `Send Transaction` again until it got successfull</i>

![image](https://user-images.githubusercontent.com/110814039/204133768-c185b19f-fa52-4265-b90f-7511b4c2256f.png)
<p align="center"> <i>this is fail transactions, if there is no Red Tag , it means succesfull</i> </p>

<b>Tips:</b> <i>change the fee on on your wallet before accept the transaction. like 0.1 until 1  </i>

## 14. Submit to Form
1. Open https://fisz9c4vvzj.typeform.com/zkSparkTutorial
2. Fill some information that they need
3. Dont forget to give `Feedback` for the team so they can improve their apps

<p align="center"><b> DONE!</b></p>


# Uninstall the Node

```
rm -rf 04-zkapp-browser-ui
rm -rf zkapp-cli
rm -rf .npm
npm uninstall -g zkapp-cli
sudo apt-get remove nodejs
```
