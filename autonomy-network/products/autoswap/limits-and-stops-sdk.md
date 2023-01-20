# Limits & Stops SDK

### Autonomy Limit Order / Stop Loss SDK interface

#### Installation:

To install the Limit/Stop interface you can use `yarn` or `npm` in your project.

&#x20;`yarn add -D @autonomylabs/limit-orders-react`

or

`npm install --save-dev @autonomylabs/limit-orders-react`

#### **Getting Started:**

To start using the limit order and stop loss interface we have created an `AutonomyProvider`. You can easily wrap your app with the provider and pass the autonomy reducers into your redux store.

```
import { configureStore } from "@reduxjs/toolkit";
import { save, load } from "redux-localstorage-simple";
import { autonomyReducers } from "@autonomylabs/limit-orders-react";

const PERSISTED_KEYS: string[] = ["your_keys"];
const store = configureStore({
  reducer: {
    ...your_reducers,
    // Pass the autonomy reducers
    ...autonomyReducers,
  },
  middleware: [save({ states: PERSISTED_KEYS, debounce: 1000 })],
  preloadedState: load({ states: PERSISTED_KEYS }),
});
export default store;
```

You can then go to your main file and wrap your app with the `AutonomyProvider`.

```
import React from "react";
import ReactDOM from "react-dom";
import { AutonomyProvider } from "@autonomylabs/limit-orders-react";
import { useActiveWeb3React } from "hooks/web3";
import { injected } from "connectors";

function Autonomy({ children }: { children?: React.ReactNode }) {
  const { library, chainId, account, activate } = useActiveWeb3React();
  return (
    <AutonomyProvider
      library={library}
      chainId={chainId}
      account={account ?? undefined}
      // Optionally pass the handler to be able to connect to a wallet via the component button
      onConnectWallet={() => activate(injected)}
      // By default `useDefaultTheme` is set to true, will use Autonomy SDK default theme if set
      useDefaultTheme={true}
      // By default `useDarkMode` is set to true
      usDarkMode={true}
      // Optionally pass any DEX router information to be used in the sdk. By default ApeSwap router will be used on BSC, and TraderJoe will be used on Avalanche.
      routerInfo={{
        routerAddress: "",
        factoryAddress: "",
        initCodeHash: "",
      }}
    >
      {children}
    </AutonomyProvider>
  );
}

ReactDOM.render(
  <StrictMode>
    <FixedGlobalStyle />
    <Web3ReactProvider getLibrary={getLibrary}>
      <Web3ProviderNetwork getLibrary={getLibrary}>
        <Provider store={store}>
          <ThemeProvider>
            <ThemedGlobalStyle />
            <HashRouter>
              <Autonomy>
                <App />
              </Autonomy>
            </HashRouter>
          </ThemeProvider>
        </Provider>
      </Web3ProviderNetwork>
    </Web3ReactProvider>
  </StrictMode>,
  document.getElementById("root")
);
```

#### Add the Limits Orders and Stops Losses:

You can easily then pass the order type (Limit or StopLoss) to our order components, `AutonomyOrderPanel` and `AutonomyOrderHistoryPanel`.

```
import React from "react";
import { AutonomyOrderPanel, AutonomyOrderHistoryPanel } from "@autonomylabs/limit-orders-react";
export default function App() {
  return (
    <div style={{ display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center" }}>
      <AutonomyOrderPanel orderType="Limit" />
      <AutonomyOrderHistoryPanel orderType="Limit" />
    </div>
  ):
}
```

### Autonomy Limit Order / Stop Loss SDK orders

Place limit & stoploss orders on any EVM chain.

#### Installation:

`yarn add -D @autonomylabs/limit-stop-orders`

or

`npm install --save-dev @autonomylabs/limit-stop-orders`

#### Initialize the SDK:

```
import { AutonomyLimitStopOrders } from "@autonomylabs/limit-stop-orders";

const autonomyLimitStopOrders = new AutonomyLimitStopOrders(
  chainId, // BSC = 56, Avalanche = 43114
  signerOrProvider, // Web3 Signer or Provider
  routerAddress, // DEX router address, Apeswap by default on BSC / TraderJoe by default on Avalanche
  factoryAddress, // DEX factory adderss, Apeswap by default on BSC / TraderJoe by default on Avalanche
  initCodeHash // DEX init code hash, Apeswap by default on BSC / TraderJoe by default on Avalanche
);
```

#### Approve Tokens:

To create limit orders and stop losses you must first approve the tokens on the Autonomy router.

```
await autonomyLimitStopOrders.approve(
  inputToken, // input token address
  inputAmount, // input amount in BigNumber
  recipient // recipient address
);
```

#### Submit an Order:

```
await autonomyLimitStopOrders.submitOrder(
  orderType, // Limit or Stop
  inputToken, // input token address
  inputAmount, // input amount in BigNumber
  outputToken, // output token address
  outputAmount, // output amount in BigNumber
  recipient, // recipient address
  autonomyPrepay // if true, small amount will be pre-paid
);
```

#### Cancel an Order:

```
await autonomyLimitStopOrders.cancelOrder(order);
```

#### Order History:

Get all Orders:

```
const allOrders = await autonomyLimitStopOrders.getOrders(requesterAddress);
```

Get open Orders:&#x20;

```
const openOrders = await autonomyLimitStopOrders.getOpenOrders(requesterAddress);
```

Get executed Orders:

```
const executedOrders = await autonomyLimitStopOrders.getExecutedOrders(requesterAddress);
```

Get cancelled Orders:

```
const cancelledOrders = await autonomyLimitStopOrders.getCancelledOrders(requesterAddress);
```

Get Orders by type:

```
const limitOrders = await autonomyLimitStopOrders.getOrdersByType(requesterAddress, "Limit");
const stopLossOrders = await autonomyLimitStopOrders.getOrdersByType(requesterAddress, "Stop");
```

### Reference integrations

Coming soon

\


\
