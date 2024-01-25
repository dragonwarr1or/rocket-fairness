# Verify Space Odyssey Fair

This game ensures fairness through a dynamic process involving a **Server Seed** and a **Client Seed**. The Server Seed, initially provided by us, undergoes multiple hashing iterations, generating a sequence of 4 million server seeds. Each game uses a server seed from this sequence, creating a transparent and verifiable gaming environment.

The Client Seed, a public key agreed upon before revelation, is based on the hash of a TON block. This commitment to the hash occurs before the block is mined, ensuring that outcomes are beyond our control. The current Client Seed is presented below.

To verify fairness, input the Server Seed into the provided code. You'll witness the server seeds and game results for the last 100 games.

Javascript
```
const crypto = require("crypto");

// Write Last Game Hash Here
const lastGameHash = "";

// Client Seed: hash of Ton Block https://tonviewer.com/transaction/ifAlY3N7Mw52d3-84FS31wZjzar1IurGp3OcAu0l5VE=
const spaceOdysseyClientSeed =
  "ifAlY3N7Mw52d3-84FS31wZjzar1IurGp3OcAu0l5VE";

// Function to generate hash from a given seed
const generateHash = (seed) =>
  crypto.createHash("sha256").update(seed).digest("hex");

// Function to check if a hash is divisible by a given modulo
const isHashDivisible = (hash, mod) => {
  let value = 0;
  const offset = hash.length % 4;

  for (let i = offset > 0 ? offset - 4 : 0; i < hash.length; i += 4) {
    value = ((value << 16) + parseInt(hash.substring(i, i + 4), 16)) % mod;
  }

  return value === 0;
};

// Function to calculate the crash point based on the server seed and client seed
const calculateCrashPoint = (serverSeed) => {
  const hash = crypto
    .createHmac("sha256", serverSeed)
    .update(spaceOdysseyClientSeed)
    .digest("hex");

  const divisibleFactor = parseInt(100 / 4);
  if (isHashDivisible(hash, divisibleFactor)) {
    return 1;
  }

  const hashValue = parseInt(hash.slice(0, 52 / 4), 16);
  const exponent = Math.pow(2, 52);

  return Math.floor((100 * exponent - hashValue) / (exponent - hashValue)) / 100.0;
};

// Function to get the results of the previous 100 games
const getPreviousGameResults = () => {
  const previousGames = [];
  let currentGameHash = generateHash(lastGameHash);

  for (let i = 0; i < 100; i++) {
    const gameResult = calculateCrashPoint(currentGameHash);
    previousGames.push({ gameHash: currentGameHash, gameResult });
    currentGameHash = generateHash(currentGameHash);
  }

  return previousGames;
};

// Function to verify the crash outcome
const verifyCrash = () => {
  const currentGameResult = calculateCrashPoint(lastGameHash);
  const previousHundredGames = getPreviousGameResults();

  return { currentGameResult, previousHundredGames };
};

// Output verification results
console.log(verifyCrash());
```
