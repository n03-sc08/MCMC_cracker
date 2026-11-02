# MCMC Cracker

Cracks **monoalphabetic substitution ciphers** without knowing the key, using a Markov-chain Monte Carlo (MCMC) style search. It scores candidate decryptions against English letter-pair (bigram) statistics and shows the search live in a small window.

I wrote it for the 2024–25 National Cipher Challenge, where our team (I was captain) won silver.

## How it works

A substitution cipher replaces each letter with another one, so the key is a permutation of the alphabet. There are 26! ≈ 4 × 10²⁶ possible keys, which is far too many to brute-force, so the program searches the key space instead:

1. **Start** with the identity key (`abcdefghijklmnopqrstuvwxyz`).
2. **Propose** a new key by swapping two random pairs of letters in the current key.
3. **Score** the decryption each key produces. The fitness function compares the bigram frequencies of the candidate plaintext against a table of English bigram probabilities, summing the absolute differences. **Lower is better.**
4. **Accept or reject** (Metropolis-style):
   - If the new key scores better or equal, always accept it.
   - If it scores worse, accept it with a small probability `Δ / (score × 250)`, which lets the search escape local optima.
5. **Stop** when the score drops to 0.4 or below.

Because the walk moves through the space of keys using only the previous key and random proposals, it is a Markov chain, and the probabilistic acceptance step is what makes it MCMC rather than plain hill-climbing.

## Live display

While running, a pygame window shows:

- the best and current key,
- the best and current fitness score,
- the first 50 characters of the best and current decryption,
- the iteration count.

You can watch readable English emerge from the ciphertext as the search converges.

## Files

| File | Purpose |
|---|---|
| `MCMC_ALGO.py` | Main script: key proposals, accept/reject loop, pygame display. The ciphertext is stored in the `cText` variable |
| `bigram_utils.py` | Text cleaning, bigram frequency counting, the English bigram probability table and the fitness function |
| `requirements.txt` | Python dependencies |

## Installation and usage

```bash
git clone https://github.com/n03-sc08/MCMC_cracker.git
cd MCMC_cracker
pip install -r requirements.txt
python MCMC_ALGO.py
```

To crack your own message, replace the text in the `cText` variable in `MCMC_ALGO.py`. Case, spaces and punctuation are stripped automatically. Stop the program with `Ctrl+C` in the terminal.

## Performance and limitations

- Each iteration decrypts and scores two candidate keys against the whole bigram table, so it runs at roughly 25–30 iterations per second. In a test run, English fragments began to appear after around 2,000 iterations, but reaching the 0.4 stop threshold takes much longer.
- Bigram statistics work best on longer ciphertexts. Short messages give noisy scores.
- The acceptance probability is a fixed constant rather than a cooling schedule, and there are no random restarts, so a run can get stuck in a local optimum.

## Ideas for improvement

- Cache the current key's score instead of recomputing it every iteration, which would roughly halve the work
- Count bigrams once per text with `collections.Counter` instead of calling `text.count` for every letter pair
- Use quadgram scoring and simulated annealing with a cooling schedule
- Handle the window-close event cleanly

## Built with

Python 3, NLTK (bigram generation), pygame (live display)
