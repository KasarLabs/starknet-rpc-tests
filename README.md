# Integration tests

## Functional testing for Madara

:information_source: This is the location of all the typescript based tests for
Madara

This folder contains a set of functional tests designed for Madara.

It is written in typescript, using Mocha/Chai as Test framework.

### Test flow

Each group will start a dev service with the
[development spec](../node/service/src/chain_spec) before executing the tests.

### Test categories

- `test`: Tests expected to run by spawning a new dev node (~1-2 minutes)
- `smoke-test`: Tests verifying the data (consistency) on an existing chain
  (~5-20 minutes)

### Installation

```sh
npm install
```

### Run the tests

```sh
npm run test
```

and to print more information:

```sh
npm run test-with-logs
```

## Smoke tests

### Adding smoke tests

Smoke test should only contain consistency/state checks.

Testing the consistency is usually simple:

- When you have redundant information: Verify they match:
  `totalIssuance == sum(accounts.map(acc => acc.free + acc.reserved))`
- When you have conditional state: Verify the condition is valid:
  `parachainStaking.topDelegations.each(top => top.length <= parachainStaking.maxTopDelegationsPerCandidate)`
- When you expect specific state: Verify it exists: `assets.assets.length > 0`
  or `maintenanceMode.maintenanceMode == false`)

Smoke tests should **never** send an extrinsic to modify the state. They should
be split by pallet and only need 1 `describeSmokeSuite` per file.

### Running smoke tests

In order to use smoke tests, you need to provide a blockchain:

```sh
WSS_URL=wss://localhost:9944 npm run smoke-test
```

You can debug specific smoke test with `debug` library using prefix `smoke:*`:

```sh
DEBUG=smoke:* WSS_URL=wss://localhost:9944 npm run smoke-test
```

### Write Tests

### Add a new contract

- Add contract source code to `cairo-contracts/src`
- Run `starknet-compile-deprecated your_file.cairo`=> This will generate the
  necessary abi and byte code

### Verbose mode

You can also add the node's logs to the output using the `MADARA_LOG` env
variable. Ex:

```sh
MADARA_LOG="warn,rpc=trace" npm run test
```

The test script will find available ports above 20000 in order to ensure that it
doesn't conflict with any other running services.

## Debugging a Madara node

The repository contains a pre-configured debugger configuration for VSCode with
the **CodeLLDB** (`vadimcn.vscode-lldb`) extension.

Before debugging, you need to build the node with debug symbols with command
`RUSTFLAGS=-g cargo build --release` (available as a VSCode task). Then go in
the **Debug** tab in the left bar of VSCode and make sure **Launch Madara Node
(Linux)** is selected in the top dropdown. **Build & Launch Madara Node
(Linux)** will trigger the build before launching the node.

Depending on what exactly you're attempting to debug, you may need other build
configurations. The most straightforward is a debug build (omit `--release`),
but this will produce a binary which is extremely large and performs very
poorly. A `--release` build can provide some middle ground, and you may need
some or all of:

- `-g` (alias for `-C debuginfo=2`, the max)
- `-C force-frame-pointers=yes`
- `-Copt-level=0` (or 1, etc. This one has a big impact)

To launch the debug session click on the green "play" arrow next to the
dropdown. It will take some time before the node starts, but the terminal
containing the node output will appear when it is really starting. The node is
listening on ports 19931 (p2p), 19932 (rpc) and 19933 (ws).

You can explore the code and place a breakpoint on a line by left clicking on
the left of the line number. The execution will pause the next time this line is
reached. The debug toolbar contains the following buttons :

- Resume/Pause : Resume the execution if paused, pause the execution at the
  current location (pretty random) if running.
- Step over : Resume the execution until next line, or go one level up if the
  end of the current scope is reached.
- Step into : Resume the execution to go inside the immediately next function
  call if any, otherwise step to next line.
- Step out : Resume the execution until the end of the scope is reached.
- Restart : Kill the program and start a new debugging session.
- Stop : Kill the program and end debugging session.

Breakpoints stay between debugging sessions. When multiple function calls are
made on the same line, multiple step into, step out, step into, ... can be
required to go inside one of the chained calls.

When paused, content of variables is showed in the debugging tab of VSCode. Some
basic types are displayed correctly (primitive types, Vec, Arc) but more complex
types such as HashMap/BTreeMap are not "smartly" displayed (content of the
struct is shown by mapping is hidden in the complexity of the implementation).

### Running Typescript tests with a debug node

By setting the environment variable `DEBUG_MODE=true`, the Typescript tests will
not spawn its own node and instead will connect to an external node running on
ports 19931/19932/19933, which are the ports used by the debug node.

A VSCode test allow to quickly run the `test-single` test in debug mode. To run
another test, change the command in the `package.json`. Note that you should
restart the node after running one test file.
