<Button on:click={async () => {
  if (!web3authProvider)
    await connect()
  else
    await logout()
  }}>{!web3authProvider ? "Connect" : "Logout"}</Button>
<br/>
<p>{address ? `Connected: ${address}`: "Not Connected"}</p>
<p>{`name: ${user?.name}`}</p>
<p>{`email: ${user?.email}`}</p>
<p>{`verifierId: ${user?.verifierId}`}</p>
<p>{`verifier: ${user?.verifier}`}</p>
<!--<p>{JSON.stringify(cfg, null, '  ')}</p> -->

<script>
  import { browser } from '$app/environment';
  import { Button } from 'flowbite-svelte';
  import { Web3Auth } from "@web3auth/modal";
  import { TorusWalletAdapter } from "@web3auth/torus-evm-adapter";
  import { ethers } from "ethers";

  let web3authProvider=undefined;
  let provider=undefined;
  let connected=false;
  let address=undefined;
  let user=undefined;
  let cfg = {};
  let web3auth;
  let torusWalletAdapter;

  function _reset() {
    web3authProvider = undefined;
    provider = undefined;
    user = undefined;
    cfg = undefined;
    address = undefined;
  }

  async function connect() {
    if (!browser) return;
    if (web3authProvider) {console.log("already connected"); return;}

    connected = !connected;
    const resp = await fetch(`/api/providers/mumbai`);
    cfg = await resp.json();
    web3auth = new Web3Auth({
      clientId: cfg.web3auth.clientId,
      web3AuthNetwork: cfg.web3auth.web3AuthNetwork,
      chainConfig: {...cfg.chainConfig},
    });

    torusWalletAdapter = new TorusWalletAdapter({
      adapterSettings: {
        buttonPosition: "bottom-left",
      },
      loginSettings: {
        verifier: "google",
      },
      initParams: {
        buildEnv: "testing",
      },
      /*chainConfig: {
        chainNamespace: CHAIN_NAMESPACES.EIP155,
        chainId: "0x1",
        rpcTarget: "https://rpc.ankr.com/eth",
        // Avoid using public rpcTarget in production.
        // Use services like Infura, Quicknode etc
        displayName: "Ethereum Mainnet",
        blockExplorer: "https://etherscan.io",
        ticker: "ETH",
        tickerName: "Ethereum",
      },*/
      clientId: cfg.web3auth.clientId,
      sessionTime: 3600, // 1 hour in seconds
      web3AuthNetwork: cfg.web3auth.web3AuthNetwork,
    });

    await web3auth.initModal();
    web3authProvider = await web3auth.connect();
    if (!web3authProvider) { console.log("connect failed or canceled"); return;}
    provider = new ethers.providers.Web3Provider(web3authProvider);
    const signer = await provider.getSigner();
    address = await signer?.getAddress();
    console.log(`connected with address: "${address}"`);
    user = await web3auth.getUserInfo();
  }

  async function logout() {
    if (!browser) return;
    if (!web3authProvider) {console.log("not connected"); return;}
    await web3auth.logout();
    _reset();
  }
</script>
