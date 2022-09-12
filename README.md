# websnark

A fast zkSnark proof generator written in native Web Assembly.

websnark is used to generate zkSnark Proofs from the browser.

This module generates highly optimized Web Assembly modules for the low level
cryptographic primitives.

It also makes use of the Web Workers feature to parallelize the generation
of the zero knoledge proofs.

The result is a fast library with times close to libsnarks but fully compatible for
browsers.

## Usage

You just need to import the websnark.js found in the build directory.

```html
<script src="websnark.js" />
```

This library has a single javascript function:

genZKSnarkProof(witness, provingKey, cb)

cb is the callback.  If cb is undefined, then the function will return a promise.

witness is a binary buffer with all the signals in binnary format.  The buffer is packt in 32bytes Little Endian Encoded Field Elements.

You can use the tool to build the binary file from the witness.json file generated by [snarkjs](https://github.com/iden3/snarkjs).

### IMPORTANT: Please be sure you run your setup with `--protocol groth`  websnark only generates groth16 proofs!

```
node ../tools/buildwitness.js -i witness.json -o witness.bin
```

provingKey is the binary buffer with the binary representation of the proving key.

Check the tool tools/buildpkey.js to convert a proving_key.json file generated
in [snarkjs](https://github.com/iden3/snarkjs) to a proving_key.bin file that can
be used directly with this library.

```
node ../tools/buildpkey.js -i proving_key.json -o proving_key.bin
```

The result is a JSON object with pi_a, pi_b and pi_c points.

You can use the stringified version of this JSON as a proof.json in [snarkjs](https://github.com/iden3/snarkjs)


Here is a simple example of a web page that loads a key and a witness and generates the proof when the button is pressed.

```html
<html>
<header>
</header>
<script src="websnark.js"></script>
<script>

var witness;
var proving_key;

function onLoad() {

    fetch("proving_key.bin").then( (response) => {
        return response.arrayBuffer();
    }).then( (b) => {
        provingKey = b;
    });

    fetch("witness.bin").then( (response) => {
        return response.arrayBuffer();
    }).then( (b) => {
        witness = b;
    });
}

function calcProof() {
    const start = new Date().getTime();
    document.getElementById("time").innerHTML = "processing....";
    document.getElementById("proof").innerHTML = "";

    window.genZKSnarkProof(witness, provingKey).then((p)=> {
        const end = new Date().getTime();
        const time = end - start;
        document.getElementById("time").innerHTML = `Time to compute: ${time}ms`;
        document.getElementById("proof").innerHTML = JSON.stringify(p, null, 1);
    });
}

</script>
<body onLoad="onLoad()">
<h1>iden3</h1>
<h2>Zero knowledge proof generator</h2>
<button onClick="calcProof()">Test</button>
<div id="time"></div>
<pre id="proof"></pre>

</body>
</html>
```

You can test it by running a web server on the example directory

```
npm -g install http-server
cd example
http-server .
```

And then navegate to [http://127.0.0.1:8080](http://127.0.0.1:8080)

The generated proof can be cut and pasted to `example/proof` and tested with snarkjs

```
snarkjs verify
``

## Building wasm.js

```
npm run build
```

## Testing

```
npm test
```

## License

websnark is part of the iden3 project copyright 2019 0KIMS association and published with GPL-3 license. Please check the COPYING file for more details.
