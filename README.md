A [hardhat-circom](https://github.com/projectsophon/hardhat-circom) project to generate zero-knowledge circuits, proofs, and solidity verifiers for the below gate configuration:

![image](https://github.com/rishabh0111/zkSNARK_Circuit_Polygon/assets/115526596/6246ffda-ffb9-4bfa-9e3d-997c65706a37)


## Quick Start
Compile the "TheCircuit" circuit and verify it against a smart contract verifier

```
pragma circom 2.0.0;

template TheCircuit () {
   // signal inputs
   signal input a;
   signal input b;

   // signals from gates
   signal x;
   signal y;

   // final signal output
   signal output q;

   // component gates used to create custom circuit
   component andGate = AND();
   component notGate = NOT();
   component orGate = OR();

   //circuit logic 
   andGate.a <== a;
   andGate.b <== b;
   x <== andGate.out;

   notGate.in <== b;
   y <== notGate.out;

   orGate.a <== x;
   orGate.b <== y;
   q <== orGate.out;

}

template AND() {
    signal input a;
    signal input b;
    signal output out;

    out <== a*b;
}

template OR() {
    signal input a;
    signal input b;
    signal output out;

    out <== a + b - a*b;
}

template NOT() {
    signal input in;
    signal output out;

    out <== 1 + in - 2*in;
}

component main = TheCircuit();
```
### Install
`npm i`

### Compile
`npx hardhat circom` 
This will generate the **out** file with circuit intermediaries and geneate the **TheCircuitVerifier.sol** contract

### Prove and Deploy
`npx hardhat run scripts/deploy.ts`
This script does 4 things  
1. Deploys the TheCircuitVerifier.sol contract
2. Generates a proof from circuit intermediaries with `generateProof()`
3. Generates calldata with `generateCallData()`
4. Calls `verifyProof()` on the verifier contract with calldata

With two commands you can compile a ZKP, generate a proof, deploy a verifier, and verify the proof 🎉

## Configuration
### Directory Structure
**circuits**
```
├── TheCircuit
│   ├── circuit.circom
│   ├── input.json
│   └── out
│       ├── circuit.wasm
│       ├── TheCircuit.r1cs
│       ├── TheCircuit.vkey
│       └── TheCircuit.zkey
├── new-circuit
└── powersOfTau28_hez_final_12.ptau
```
Each new circuit lives in it's own directory. At the top level of each circuit directory lives the circom circuit and input to the circuit.
The **out** directory will be autogenerated and store the compiled outputs, keys, and proofs. The Powers of Tau file comes from the Polygon Hermez ceremony, which saves time by not needing a new ceremony. 


**contracts**
```
contracts
└── TheCircuitVerifier.sol
```
Verifier contracts are autogenerated and prefixed by the circuit name, in this example **Multiplier**

## hardhat.config.ts
```
  circom: {
    // (optional) Base path for input files, defaults to `./circuits/`
    inputBasePath: "./circuits",
    // (required) The final ptau file, relative to inputBasePath, from a Phase 1 ceremony
    ptau: "powersOfTau28_hez_final_12.ptau",
    // (required) Each object in this array refers to a separate circuit
    circuits: JSON.parse(JSON.stringify(circuits))
  },
```
### circuits.config.json
circuits configuation is separated from hardhat.config.ts for **autogenerated** purposes (see next section)
```
[
  {
    "name": "TheCircuit",
    "protocol": "groth16",
    "circuit": "TheCircuit/circuit.circom",
    "input": "TheCircuit/input.json",
    "wasm": "TheCircuit/out/circuit.wasm",
    "zkey": "TheCircuit/out/TheCircuit.zkey",
    "vkey": "TheCircuit/out/TheCircuit.vkey",
    "r1cs": "TheCircuit/out/TheCircuit.r1cs",
    "beacon": "0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f"
  }
]
```

