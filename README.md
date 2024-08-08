# Zero-Knowledge Proof System Setup

This guide will walk you through setting up a non-interactive zero-knowledge proof (ZKP) system using Circom and snarkJS.

## Steps to Create a ZKP System:
1. Generate the proving and verification keys.
2. Generate the proof.
3. Verify the proof.
https://res.cloudinary.com/practicaldev/image/fetch/s--IAz6JB7w--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/aoasgise8c4gjjf4q5ch.png
---

## Step 1: Generate the Proving and Verification Keys

### Prerequisites:
1. **Node.js**
2. **Circom Compiler**

### Project Setup
 bash
npm init -y
npm install circomlib@2.0.5 

* A circuit in zero-knowledge proof is a program that specifies a calculation to be performed on some data inputs. The circuit is used by the prover to generate a proof that they have correctly performed the calculation, without revealing any information about the data inputs. The verifier can then use the proof to confirm that the computation was performed correctly, without learning anything about the data inputs.

These circuits can be complex made of logic gates like (AND, OR) but fortunately we have a programming language called CIRCOM which is DSL(Domain specific language) used to create aritmetic circuits.

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
* The program defines a template called "Hasher" which takes an input signal called "secret" and outputs a signal called "out". It uses a component called "Pedersen" which is included from the circomlib library, and sets it up to use 1 input.
The code then connects the "secret" input to the first input of the "Pedersen" component, and the output of the "Pedersen" component to the "out" output of the template.
Finally, the program defines a component called "main" which is an instance of the "Hasher" template.
Overall, this program creates a circuit that takes a secret input, hashes it using the Pedersen hashing function, and outputs the resulting hash.

Compile the Circuit

```
circom circuit.circom --r1cs --wasm
```
* after compiling we get 2 files(circuit.r1cs, circuit.wasm). r1cs(rank 1 constraint system), In simple terms, it is a way to express any computation as a set of linear equations that satisfy certain constraints. These constraints can be thought of as rules that define the inputs and outputs of the computation. We also get a wasm binary(web assembly).

Now one part of step one is completed i.e we have a circuit and now we need a secret value(λ).
This secret value is generated with the help of a ceremony called as trusted setup.

Trusted setup is a process used in some zero knowledge proof systems to generate the initial proving and verifying keys required for generating and verifying proofs. The purpose of trusted setup is to establish a secure foundation for the system by generating these keys in a way that ensures they are unpredictable and unbiased. This is typically done by having a group of trusted individuals generate the keys together, with each individual contributing a piece of random data. Once all the pieces are combined and processed using cryptographic techniques, the resulting keys can be used to generate and verify proofs without the need for any additional trust assumptions. The goal of trusted setup is to ensure that the system is secure and trustworthy, even if some of the participants in the trusted setup process are malicious.
Generate the Secret Value (λ)
```
wget https://hermez.s3-eu-west-1.amazonaws.com/powersOfTau28_hez_final_12.ptau
npm install snarkjs
```
Generate Keys
```
npx snarkjs groth16 setup circuit.r1cs powersOfTau28_hez_final_12.ptau circuit_0000.zkey
```
circuit_0000.zkey is a binary file that contains the proving and verification keys.


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
* Proof(π) is successfully generated and now we need to generate the verification key from the proving key(circuit_0000.zkey).

Proving key is like private key and verification key is like public key(In context of assymetric key cryptography)
```
npx snarkjs zkey export verificationkey circuit_0000.zkey verification_key.json
```
## Step 3: Verify the Proof
* Verifying the Proof(π) which is generated in step 2.

Note: We can write code to verify the proof on a normal backend or we can also do on-chain verification with the help of smart contract.

let's create web2 backend verification system first.

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

https://dev.to/yagnadeepxo/the-beginners-guide-to-zk-snark-setting-up-your-first-proof-system-3le3
