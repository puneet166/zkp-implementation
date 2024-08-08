# Zero-Knowledge Proof System Setup

This guide will walk you through setting up a non-interactive zero-knowledge proof (ZKP) system using Circom and snarkJS.

## Steps to Create a ZKP System:
1. Generate the proving and verification keys.
2. Generate the proof.
3. Verify the proof.

---

## Step 1: Generate the Proving and Verification Keys

### Prerequisites:
1. **Node.js**
2. **Circom Compiler**

### Project Setup
 bash
npm init -y
npm install circomlib@2.0.5 

## Write the Circuit
Create circuit.circom:

```
pragma circom 2.0.0;

include "node_modules/circomlib/circuits/pedersen.circom";

template Hasher(){
    signal input secret;
    signal output out;

    component pedersen = Pedersen(1);
    pedersen.in[0] <== secret;
    out <== pedersen.out[0];
}

component main = Hasher();
```
Compile the Circuit

```
circom circuit.circom --r1cs --wasm
```
Generate the Secret Value (λ)
```
wget https://hermez.s3-eu-west-1.amazonaws.com/powersOfTau28_hez_final_12.ptau
npm install snarkjs
```
Generate Keys
```
npx snarkjs groth16 setup circuit.r1cs powersOfTau28_hez_final_12.ptau circuit_0000.zkey
```
## Step 2: Generate the Proof

Create index.js:

```
const snarkjs = require('snarkjs');

async function generateProof(){
    const { proof, publicSignals } = await snarkjs.groth16.fullProve(
        { secret: 12345 }, 
        "circuit_js/circuit.wasm", 
        "circuit_0000.zkey"
    );
    console.log(publicSignals);
    console.log(proof);
}

generateProof().then(() => {
    process.exit(0);
});
```
Generate the Verification Key
```
npx snarkjs zkey export verificationkey circuit_0000.zkey verification_key.json
```
## Step 3: Verify the Proof

Create verify.js:

```
const snarkjs = require('snarkjs');
const fs = require("fs");

async function verify(){
    const { proof, publicSignals } = await snarkjs.groth16.fullProve(
        { secret: 12345 }, 
        "circuit_js/circuit.wasm", 
        "circuit_0000.zkey"
    );

    const vKey = JSON.parse(fs.readFileSync("verification_key.json"));
    const res = await snarkjs.groth16.verify(vKey, publicSignals, proof);

    if (res === true) {
        console.log("Verification OK");
    } else {
        console.log("Invalid proof");
    }
}

verify().then(() => {
    process.exit(0);
});
```
Run the script to verify the proof. If everything is correct, you should see:

```
Verification OK
```
🎉 Congratulations! You've completed all three steps!

