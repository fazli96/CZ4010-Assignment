# CZ4010-Assignment
Linear Cryptanalysis on SPN cipher

# Objective

The objective of this project is to implement a linear cryptanalysis attack against a
Substitution-Permutation Network (SPN) in order to recover at least all the bits of the final round key. The SPN network used for this project as shown in Figure 1 consists of only 4 rounds with each Substitution-Box (S-box) being the same across all the rounds. Each round consists of substitution, a transposition of the bits (i.e., permutation of the bit positions), and key mixing.

# Development

This project has been developed using Google Colab with Python 3.6.9. The main file in this project is

* CZ4010_Assignment.ipynb


> Implements the SPN cipher including key generation, encryption and decryption methods. The cipher takes as input a 16-bit input block and has 4 rounds, each comprising substitution, transposition and key mixing. Running this file will:

>* Initialize the SPN cipher with S-box mapping and P-box mapping
>* Generate round keys from a master key using a key scheduling algorithm
>* Encrypt 20,000 random values using round keys generated
>* Print the Linear Approximation Table for the SPN S-box
>* Recover all 16 bits of the final round key using linear cryptanalytic techniques
>* Recover the rest of the round keys (1st, 2nd, 3rd, 4th rounds) using linear cryptanalytic techniques

# Running the code

There are two ways of running the code, one being using Google Colab and the other using Jupyter Notebook.

* Google Colab

> Clone the repository into a folder in your local machine and upload this folder into your Google Drive. Once uploaded, run CZ4010_Assignment.ipynb in Google Colab.

* Jupyter Notebook

> Clone the repository into a folder in your local machine. Open Jupyter notebook and run CZ4010_Assignment.ipynb located in that folder.

# Cipher

![SPN Cipher](https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/SPN_Cipher.png?raw=true)

> Figure 1: Substitution-Permutation Network (SPN)

## Substitution

In our cipher, we break the 16-bit data block into four 4-bit sub-blocks. Each sub-block forms an input to a 4×4 S-box (a substitution with 4 input and 4 output bits), which is a nonlinear mapping, where the output bits cannot be represented as a linear operation on the input bits.The mapping chosen for our S-box, is given in Table 1, where the most significant bit of the hexadecimal notation represents the leftmost bit of the S-box. The S-box mapping is illustrated in the functions sbox_map (input to output) and sbox_inverse_map (output to input). Encryption of a 16-bit data block using the 4 S-boxes is illustrated in the function sbox_encrypt(state) whereas its decryption is illustrated in the function sbox_decrypt(state).

![Sbox Mapping](https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Sbox_Mapping.png?raw=true)

> Table 1: S-box mapping

## Permutation

The permutation portion of a round is simply the transposition of the bits or the Permutation-Box (P-box). The permutation of our P-box is given in Table 2 where the numbers represent bit positions in the block, with 1 being the leftmost bit and 16 being the rightmost bit and can be simply described as: the output i of S-box<sub>j</sub> is connected to input j of S-box<sub>i</sub>. In our cipher, the last round does not have a P-box. The P-box mapping is illustrated in the function pbox_map. As P-box is simply the transposition of the bits, both encryption and decryption of a 16-bit data block using the 4 S-boxes is illustrated in the function pbox(state).

![Pbox Mapping](https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Pbox_Mapping.png?raw=true)

> Table 2: P-box mapping

## Key Mixing

To achieve the key mixing, we use a simple bitwise exclusive-OR between the key bits associated with a round (referred to as a subkey) and the data block input to a round. A subkey is also applied following the last round, ensuring that the last layer of substitution cannot be easily ignored by a cryptanalyst that simply works backward through the last round’s substitution. The subkey for a round is derived from the cipher’s master key through a process known as the key schedule. The key schedule algorithm used for the key mixing is adapted from https://eprint.iacr.org/2020/1545 and illustrated in the function key_scheduler_algo(master_key, num_rounds, alpha = 13, gamma = 8). The following is the round keys generated with master key = 0x6FF91

* Round Key #1: 0x6ff9
* Round Key #2: 0xd2de
* Round Key #3: 0x9fa7
* Round Key #4: 0x773c
* Round Key #5: 0x38ea

## Encryption and Decryption

In order to encrypt, data is passed forward through the SPN network undergoing substitution, permutation and key mixing. The encryption of a plaintext, a 16-bit data block, is illustrated in the function encrypt(pt, k) where pt is the plaintext and k is all the round keys combined together.

Generating n pairs of plaintext and ciphertext is illustrated in the function generate_plaintext_ciphertext_pairs(num_of_plaintext_ciphertext_pairs)

In order to decrypt, data is passed backwards through the SPN network. However, the mappings used in the S-boxes of the decryption network are the inverse of the mappings
in the encryption network. This implies that in order for an SPN to allow for decryption, all S-boxes must be bijective, that is, a one-to-one mapping with the same number input and output bits. The round keys are also applied in reverse order and the bits of the round keys must be moved around according to the P-box mapping. The decryption of a ciphertext is illustrated in the function decrypt(ct, k) where ct is the ciphertext and k is all the round keys combined together.

# Linear Cryptanalysis

## Overview

Linear cryptanalysis tries to take advantage of high probability occurrences of linear expressions involving plaintext bits, "ciphertext" bits (actually we shall use bits from the 2nd last round output), and subkey bits. It is a known plaintext attack: that is, it is premised on the attacker having information on a set of plaintexts and the corresponding ciphertexts. However, the attacker has no way to select which plaintexts (and corresponding ciphertexts) are available. In many applications and scenarios it is reasonable to assume that the attacker has knowledge of a random set of plaintexts and the corresponding ciphertexts. The basic idea is to approximate the operation of a portion of the cipher with an expression that is linear where the linearity refers to a mod-2 bit-wise operation (i.e., exclusive-OR denoted by "⊕"). Such an expression is of the form:


X<sub>i1</sub> ⊕ X<sub>i2</sub> ⊕ ... ⊕ X<sub>iu</sub> ⊕ Y<sub>i1</sub> ⊕ Y<sub>i2</sub> ⊕ ... ⊕ Y<sub>iu</sub> = 0 (Equation 1)


where X<sub>i</sub> represents the i-th bit of the input X = [X<sub>1</sub>, X<sub>2</sub>, ...] and Y<sub>j</sub>} represents the j-th bit of the output Y = [Y<sub>1</sub>, Y<sub>2</sub>, ...]. This equation is representing the exclusive-OR "sum" of u input bits and v output bits. The approach in linear cryptanalysis is to determine expressions of the form above which have a high or low probability of occurrence. (No obvious linearity such as above should hold for all input and output values or the cipher would be trivially weak.) If a cipher displays a tendency for (Equation 1) to hold with high probability or not hold with high probability, this is evidence of the cipher’s poor randomization abilities. Consider that if we randomly selected values for u + v bits and placed them into the equation above, the probability that the expression would hold would be exactly 1/2. It is the deviation or bias from the probability of 1/2 for an expression to hold that is exploited in linear cryptanalysis: the further away that a linear expression is from holding with a probability of 1/2, the better the cryptanalyst is able to apply linear cryptanalysis.

## Piling-Up Principle

The Piling-Up Lemma proposed by Matsui can be used to determine the bias for the linear approximation of a cipher. The Piling-Up Lemma can be simplified to the equation below:

![Piling-Up Lemma](https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Piling_Up_Lemma.png?raw=true)

where ε<sub>1,2,..,n</sub> represents the bias of X<sub>1</sub> ⊕ ... ⊕ X<sub>n</sub> = 0 for n independent, random binary variables, X<sub>1</sub>, X<sub>2</sub>, ...X<sub>n</sub>.

The Piling up Lemma is illustrated in the function piling_up_lemma(bias_list) where the bias list is a list of biases in all linear approximations of S-box used to mount the linear cryptanalysis attack.

## Linear Approximations of S-box

Before considering the linear cryptanalysis attack on the cipher, we must know the linear vulnerabilities of an S-box by finding the probability bias of the linear approximation of all 16 possible input values of X and their corresponding output values of Y. For example in X<sub>2</sub> ⊕ X<sub>3</sub> = Y<sub>1</sub> ⊕ Y<sub>3</sub> ⊕ Y<sub>4</sub>, it can be observed that for exactly 12 out of the 16 cases , the expression holds true. Hence, the probability bias is 12/16 -1/2 = 1/4. A complete enumeration of all linear approximations of the S-box in our cipher is given in the linear approximation table of Table 3. Each element in the table represents the number of matches between the linear equation represented in hexadecimal as "Input Sum" and the sum of the output bits represented in hexadecimal as "Output Sum" minus 8. Hence, dividing an element value by 16 gives the probability bias for the particular linear combination of input and output bits. Table 3 is generated using the function generate_linear_approximation_table().

![Linear Approximation Table](https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Approx_Table.png?raw=true)

> Table 3: Linear Approximation Table

## Complexity of Linear Cryptanalysis

In Matsui’s paper, he shows that the number of known plaintexts required in the attack is proportional to ε^<sup>−2</sup>. Hence we can approximate the number of known plaintexts required, NL, as NL ≈ 1/(ε<sup>2</sup>) where ε represents the bias from 1/2 of the probability that the linear expression for the complete cipher holds.

# Obtaining the final round key

## Linear Approximations used to recover partial subkey values [K<sub>5,5</sub>...K<sub>5,8</sub>] and [K<sub>5,13</sub>...K<sub>5,16</sub>]

<img src="https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Trail_5th_Round_Key_1.png" width="30%">

We use the following approximations of the S-box:

> S<sub>12</sub>: X<sub>1</sub> ⊕ X<sub>3</sub> ⊕ X<sub>4</sub> = Y<sub>2</sub> with probability 12/16 and bias +1/4

> S<sub>22</sub>: X<sub>2</sub> = Y<sub>2</sub> ⊕ Y<sub>4</sub> with probability 4/16 and bias −1/4

> S<sub>32</sub>: X<sub>2</sub> = Y<sub>2</sub> ⊕ Y<sub>4</sub> with probability 4/16 and bias −1/4

> S<sub>34</sub>: X<sub>2</sub> = Y<sub>2</sub> ⊕ Y<sub>4</sub> with probability 4/16 and bias −1/4

Letting U<sub>i</sub> (V<sub>i</sub>) represent the 16-bit block of bits at the input (output) of the round i of S-boxes and U<sub>i,j</sub> (V<sub>i,j</sub>) represent the j-th bit of block U<sub>i</sub> (V<sub>i</sub>) (where bits are numbered from 1 to 16 from left to right in the figure of the cipher). Similarly, let K<sub>i</sub> represent the subkey block of bits exclusive-ORed at the input to round i, with the exception that K<sub>5</sub> is the key exclusive-ORed at the output of round 4. 

Combining the linear approximations above, we get

> U<sub>4,6</sub> ⊕ U<sub>4,8</sub> ⊕ U<sub>4,14</sub> ⊕ U<sub>4,16</sub> ⊕ P<sub>5</sub> ⊕ P<sub>7</sub> ⊕ P<sub>8</sub> ⊕ ΣK = 0

where 

> ΣK = K<sub>1,5</sub> ⊕ K<sub>1,7</sub> ⊕ K<sub>1,8</sub> ⊕ K<sub>2,6</sub> ⊕ K<sub>3,6</sub> ⊕ K<sub>3,14</sub> ⊕ K<sub>4,6</sub> ⊕ K<sub>4,8</sub> ⊕ K<sub>4,14</sub> ⊕ K<sub>4,16</sub>

and ΣK is fixed at either 0 or 1 depending on the key of the cipher. By application of the Piling-Up Lemma, the above expression holds with probability 

> 1/2+2^3 (3/4−1/2)(1/4−1/2)^3 = 15/32 (that is, with a bias of −1/32).

> No of plaintext-ciphertext pairs required >= 1/(-1/32)^2 = 1024 

Now since ΣK is fixed, we note that 

> U<sub>4,6</sub> ⊕ U<sub>4,8</sub> ⊕ U<sub>4,14</sub> ⊕ U<sub>4,16</sub> ⊕ P<sub>5</sub> ⊕ P<sub>7</sub> ⊕ P<sub>8</sub> = 0 (Equation 2)

must hold with a probability of either 15/32 or (1−15/32) = 17/32, depending on whether ΣK = 0 or 1, respectively. In other words, we now have a linear approximation of the first three rounds of the cipher with a bias of magnitude 1/32.

The linear expression of Equation 2 affects the inputs to S-boxes S<sub>42</sub> and S<sub>44</sub> in the last round. For each plaintext/ciphertext sample, we would try all 256 values for the target partial subkey [K<sub>5,5</sub>...K<sub>5,8</sub>, K<sub>5,13</sub>...K<sub>5,16</sub>]. For each partial subkey value, we would increment the count whenever Equation 2 holds true, where we determine the value of [U<sub>4,5</sub>...U<sub>4,8</sub>, U<sub>4,13</sub>...U<sub>4,16</sub>] by running the data backwards through the target partial subkey and S-boxes S<sub>42</sub> and S<sub>44</sub>. The count which deviates the largest from half of the number of plaintext/ciphertext samples is assumed to the correct value. Whether the deviation is positive or negative will depend on the values of the subkey bits involved in ΣK. When ΣK = 0, the linear approximation of (5) will serve as the estimate (with probability < 1/2) and when ΣK = 1, (5) will hold with a probability > 1/2.

We have simulated attacking our basic cipher by generating 10000 known plaintext/ciphertext values and using linear cryptanalysis, the partial subkey [K<sub>5,5</sub>...K<sub>5,8</sub>] and [K<sub>5,13</sub>...K<sub>5,16</sub>] with the largest bias magnitude is 0x8 and 0xA which corresponded to the target partial subkey value [0x8, 0xA], confirming that the attack has successfully derived the subkey bits. The bias magnitude is derived from |bias| = |count − 5000| / 10000 where the count is the count corresponding to the particular partial subkey value.

## Linear Approximations used to recover partial subkey values [K<sub>5,1</sub>...K<sub>5,4</sub>] and [K<sub>5,9</sub>...K<sub>5,12</sub>]

<img src="https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Trail_5th_Round_Key_2.png" width="30%">

We use the following approximations of the S-box:

> S<sub>12</sub>: X<sub>1</sub> ⊕ X<sub>3</sub> ⊕ X<sub>4</sub> = Y<sub>2</sub> with probability 12/16 and bias +1/4

> S<sub>22</sub>: X<sub>2</sub> = Y<sub>2</sub> ⊕ Y<sub>4</sub> with probability 4/16 and bias −1/4

> S<sub>32</sub>: X<sub>2</sub> = Y<sub>1</sub> ⊕ Y<sub>2</sub> ⊕ Y<sub>3</sub> with probability 10/16 and bias +1/8

> S<sub>34</sub>: X<sub>2</sub> = Y<sub>1</sub> ⊕ Y<sub>2</sub> ⊕ Y<sub>3</sub> with probability 10/16 and bias +1/8

Combining the linear approximations above, we get

> U<sub>4,2</sub> ⊕ U<sub>4,4</sub> ⊕ U<sub>4,6</sub> ⊕ U<sub>4,8</sub> ⊕ U<sub>4,10</sub> ⊕ U<sub>4,12</sub> ⊕ P<sub>5</sub> ⊕ P<sub>7</sub> ⊕ P<sub>8</sub> ⊕ ΣK = 0 (Equation 3)

where 

> ΣK = K<sub>1,5</sub> ⊕ K<sub>1,7</sub> ⊕ K<sub>1,8</sub> ⊕ K<sub>2,6</sub> ⊕ K<sub>3,6</sub> ⊕ K<sub>3,14</sub> ⊕ K<sub>4,4</sub> ⊕ K<sub>4,6</sub> ⊕ K<sub>4,8</sub> ⊕ K<sub>4,10</sub> ⊕ K<sub>4,12</sub> ⊕ K<sub>4,14</sub>

and ΣK is fixed at either 0 or 1 depending on the key of the cipher. By application of the Piling-Up Lemma, the above expression holds with probability 

> 1/2+2^3 (3/4−1/2)(1/4-1/2)(5/8−1/2)^2 = 63/128 (that is, with a bias of −1/128).

> No of plaintext-ciphertext pairs required >= 1/(-1/128)^2 = 16384 

As the bias is much smaller than the bias obtained when recovering the partial subkey values [K<sub>5,5</sub>...K<sub>5,8</sub>] and [K<sub>5,13</sub>...K<sub>5,16</sub>], the number of plaintext-ciphertext pairs used is much greater. The linear expression of Equation 3 affects the inputs to S-boxes S<sub>41</sub>, S<sub>42</sub> and S<sub>43</sub> in the last round. Since the partial subkey values [K<sub>5,5</sub>...K<sub>5,8</sub>] has been obtained, for each plaintext/ciphertext sample, we would try all 256 values for the target partial subkey [K<sub>5,1</sub>...K<sub>5,4</sub>, K<sub>5,9</sub>...K<sub>5,12</sub>]. For each partial subkey value, we would increment the count whenever Equation 3 holds true, where we determine the value of [U<sub>4,1</sub>...U<sub>4,4</sub>], [U<sub>4,5</sub>...U<sub>4,8</sub>] and [U<sub>4,9</sub>...U<sub>4,12</sub>] by running the data backwards through target partial subkey and S-boxes S41, S42 and S43. Hence, we have simulated attacking our basic cipher by generating 20000 known plaintext/ciphertext values and using linear cryptanalysis, the partial subkey values [K<sub>5,1</sub>...K<sub>5,4</sub>] and [K<sub>5,9</sub>...K<sub>5,12</sub>] with the largest bias magnitude is 0x3 and 0xE which corresponded to the target partial subkey value [0x3, 0xE], confirming that the attack has successfully derived the subkey bits.

Thus, we managed to recover the 5th round key = 0x38ea

# Obtaining the 4th round key

## Linear Approximations used to recover partial subkey values [K<sub>4,2</sub>, K<sub>4,4</sub>, K<sub>4,6</sub>, K<sub>4,8</sub>, K<sub>4,10</sub>, K<sub>4,12</sub>, K<sub>4,14</sub>, K<sub>4,16</sub>]

<img src="https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Trail_4th_Round_Key_1.png" width="30%">

We use the following approximations of the S-box:

> S<sub>12</sub>: X<sub>1</sub> ⊕ X<sub>3</sub> ⊕ X<sub>4</sub> = Y<sub>2</sub> with probability 12/16 and bias +1/4

> S<sub>22</sub>: X<sub>2</sub> = Y<sub>2</sub> ⊕ Y<sub>4</sub> with probability 4/16 and bias −1/4

Combining the linear approximations above, we get

> U<sub>3,6</sub> ⊕ U<sub>3,14</sub> ⊕ P<sub>5</sub> ⊕ P<sub>7</sub> ⊕ P<sub>8</sub> ⊕ ΣK = 0 (Equation 4)

where 

> ΣK = K<sub>1,5</sub> ⊕ K<sub>1,7</sub> ⊕ K<sub>1,8</sub> ⊕ K<sub>2,6</sub> ⊕ K<sub>3,6</sub> ⊕ K<sub>3,14</sub>

and ΣK is fixed at either 0 or 1 depending on the key of the cipher. By application of the Piling-Up Lemma, the above expression holds with probability 

> 1/2+2(3/4−1/2)(1/4-1/2) = 3/8 (that is, with a bias of −1/8).

> No of plaintext-ciphertext pairs required >= 1/(-1/8)^2 = 64 

As the bias is much larger than the bias obtained when recovering the partial subkey values [K<sub>5,5</sub>...K<sub>5,8</sub>] and [K<sub>5,13</sub>...K<sub>5,16</sub>], the number of plaintext-ciphertext pairs used is much smaller. The linear expression of Equation 4 affects the inputs to S-boxes S<sub>32</sub> and S<sub>34</sub> in the 3rd round and the outputs of these S-boxes correspond to the even bits of the 4th round key. For each partial subkey value, we would increment the count whenever Equation 4 holds true, where we determine the value of [U<sub>3,5</sub>...U<sub>3,8</sub>] and [U<sub>3,13</sub>...U<sub>3,16</sub>] by running the data backwards through the final round of SPN, target partial subkey and S-boxes S32 and S34. Hence, for each plaintext/ciphertext sample, we would try all 256 values for the target partial subkey [K<sub>4,2</sub>, K<sub>4,4</sub>, K<sub>4,6</sub>, K<sub>4,8</sub>, K<sub>4,10</sub>, K<sub>4,12</sub>, K<sub>4,14</sub>, K<sub>4,16</sub>]. Hence, we have simulated attacking our basic cipher by generating 1000 known plaintext/ciphertext values and using linear cryptanalysis, the partial subkey values [K<sub>4,2</sub>, K<sub>4,4</sub>, K<sub>4,6</sub>, K<sub>4,8</sub>, K<sub>4,10</sub>, K<sub>4,12</sub>, K<sub>4,14</sub>, K<sub>4,16</sub>] with the largest bias magnitude is 11110110<sub>2</sub> which corresponded to the target partial subkey value [11110110<sub>2</sub>], confirming that the attack has successfully derived the subkey bits.

## Linear Approximations used to recover partial subkey values [K<sub>4,1</sub>, K<sub>4,3</sub>, K<sub>4,5</sub>, K<sub>4,7</sub>, K<sub>4,9</sub>, K<sub>4,11</sub>, K<sub>4,13</sub>, K<sub>4,15</sub>]

<img src="https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Trail_4th_Round_Key_2.png" width="30%">

We use the following approximations of the S-box:

> S<sub>12</sub>: X<sub>1</sub> ⊕ X<sub>3</sub> ⊕ X<sub>4</sub> = Y<sub>2</sub> with probability 12/16 and bias +1/4

> S<sub>22</sub>: X<sub>2</sub> = Y<sub>1</sub> ⊕ Y<sub>2</sub> ⊕ Y<sub>3</sub> with probability 10/16 and bias +1/8

Combining the linear approximations above, we get

> U<sub>3,2</sub> ⊕ U<sub>3,6</sub> ⊕ U<sub>3,10</sub> ⊕ P<sub>5</sub> ⊕ P<sub>7</sub> ⊕ P<sub>8</sub> ⊕ ΣK = 0 (Equation 5)

where 

> ΣK = K<sub>1,5</sub> ⊕ K<sub>1,7</sub> ⊕ K<sub>1,8</sub> ⊕ K<sub>2,6</sub> ⊕ K<sub>3,2</sub> ⊕ K<sub>3,6</sub> ⊕ K<sub>3,10</sub>

and ΣK is fixed at either 0 or 1 depending on the key of the cipher. By application of the Piling-Up Lemma, the above expression holds with probability 

> 1/2+2(3/4−1/2)(5/8−1/2) = 9/16 (that is, with a bias of −1/16).

> No of plaintext-ciphertext pairs required >= 1/(1/16)^2 = 256

The linear expression of Equation 5 affects the inputs to S-boxes S<sub>31</sub>, S<sub>32</sub> and S<sub>33</sub> in the 3rd round and the outputs of these S-boxes correspond to the odd bits of the 4th round key. Since the partial subkey values [K<sub>4,2</sub>, K<sub>4,6</sub>, K<sub>4,10</sub>, K<sub>4,14</sub>] has been obtained, for each plaintext/ciphertext sample, we would try all 256 values for the target partial subkey [K<sub>4,1</sub>, K<sub>4,3</sub>, K<sub>4,5</sub>, K<sub>4,7</sub>, K<sub>4,9</sub>, K<sub>4,11</sub>, K<sub>4,13</sub>, K<sub>4,15</sub>]. For each partial subkey value, we would increment the count whenever Equation 5 holds true, where we determine the value of [U<sub>3,1</sub>...U<sub>3,4</sub>], [U<sub>3,5</sub>...U<sub>3,8</sub>] and [U<sub>3,9</sub>...U<sub>3,12</sub>] by running the data backwards through the final round of SPN, target partial subkey and S-boxes S31, S32 and S33. Hence, we have simulated attacking our basic cipher by generating 5000 known plaintext/ciphertext values and using linear cryptanalysis, the partial subkey values [K<sub>4,1</sub>, K<sub>4,3</sub>, K<sub>4,5</sub>, K<sub>4,7</sub>, K<sub>4,9</sub>, K<sub>4,11</sub>, K<sub>4,13</sub>, K<sub>4,15</sub>] with the largest bias magnitude is 01010110<sub>2</sub> which corresponded to the target partial subkey value [01010110<sub>2</sub>], confirming that the attack has successfully derived the subkey bits. 

Thus, we managed to recover the 4th round key = 0x773c

# Obtaining the 3rd round key

## Linear Approximations used to recover partial subkey values [K<sub>3,2</sub>, K<sub>3,4</sub>, K<sub>3,6</sub>, K<sub>3,8</sub>, K<sub>3,10</sub>, K<sub>3,12</sub>, K<sub>3,14</sub>, K<sub>3,16</sub>]

<img src="https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Trail_3rd_Round_Key_1.png" width="30%">

We use the following approximations of the S-box:

> S<sub>12</sub>: X<sub>2</sub> = Y<sub>2</sub> ⊕ Y<sub>4</sub> with probability 4/16 and bias −1/4

Combining the linear approximations above, we get

> U<sub>2,6</sub> ⊕ U<sub>2,14</sub> ⊕ P<sub>6</sub> ⊕ ΣK = 0 (Equation 6)

where 

> ΣK = K<sub>1,5</sub> ⊕ K<sub>1,7</sub> ⊕ K<sub>1,8</sub> ⊕ K<sub>2,6</sub> ⊕ K<sub>2,14</sub>

and ΣK is fixed at either 0 or 1 depending on the key of the cipher. By application of the Piling-Up Lemma, the above expression holds with probability 

> 1/2+(1/4−1/2) = 1/4 (that is, with a bias of −1/4).

> No of plaintext-ciphertext pairs required >= 1/(-1/4)^2 = 16 

As the bias is much larger than the bias obtained when recovering the 4th round key values, the number of plaintext-ciphertext pairs used is much smaller. The linear expression of Equation 6 affects the inputs to S-boxes S<sub>22</sub> and S<sub>24</sub> in the 2nd round and the outputs of these S-boxes correspond to the even bits of the 3rd round key. For each partial subkey value, we would increment the count whenever Equation 6 holds true, where we determine the value of [U<sub>2,5</sub>...U<sub>2,8</sub>] and [U<sub>2,13</sub>...U<sub>2,16</sub>] by running the data backwards through the 3rd and final round of SPN, target partial subkey and S-boxes S22 and S24. Hence, for each plaintext/ciphertext sample, we would try all 256 values for the target partial subkey [K<sub>3,2</sub>, K<sub>3,4</sub>, K<sub>3,6</sub>, K<sub>3,8</sub>, K<sub>3,10</sub>, K<sub>3,12</sub>, K<sub>3,14</sub>, K<sub>3,16</sub>]. Hence, we have simulated attacking our basic cipher by generating 500 known plaintext/ciphertext values and using linear cryptanalysis, the partial subkey values [K<sub>3,2</sub>, K<sub>3,4</sub>, K<sub>3,6</sub>, K<sub>3,8</sub>, K<sub>3,10</sub>, K<sub>3,12</sub>, K<sub>3,14</sub>, K<sub>3,16</sub>] with the largest bias magnitude is 01110011<sub>2</sub> which corresponded to the target partial subkey value [01110011<sub>2</sub>], confirming that the attack has successfully derived the subkey bits.

## Linear Approximations used to recover partial subkey values [K<sub>3,1</sub>, K<sub>3,3</sub>, K<sub>3,5</sub>, K<sub>3,7</sub>, K<sub>3,9</sub>, K<sub>3,11</sub>, K<sub>3,13</sub>, K<sub>3,15</sub>]

<img src="https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Trail_3rd_Round_Key_2.png" width="30%">

We use the following approximations of the S-box:

> S<sub>12</sub>: X<sub>2</sub> = Y<sub>1</sub> ⊕ Y<sub>2</sub> ⊕ Y<sub>3</sub> with probability 10/16 and bias +1/8

Combining the linear approximations above, we get

> U<sub>2,2</sub> ⊕ U<sub>2,6</sub> ⊕ U<sub>2,10</sub> ⊕ P<sub>6</sub> ⊕ ΣK = 0 (Equation 7)

where 

> ΣK = K<sub>1,5</sub> ⊕ K<sub>1,7</sub> ⊕ K<sub>1,8</sub> ⊕ K<sub>2,2</sub> ⊕ K<sub>2,6</sub> ⊕ K<sub>2,10</sub>

and ΣK is fixed at either 0 or 1 depending on the key of the cipher. By application of the Piling-Up Lemma, the above expression holds with probability 

> 1/2+(5/8−1/2) = 5/8 (that is, with a bias of 1/8).

> No of plaintext-ciphertext pairs required >= 1/(1/8)^2 = 64

The linear expression of Equation 7 affects the inputs to S-boxes S<sub>21</sub>, S<sub>22</sub> and S<sub>23</sub> in the 2nd round and the outputs of these S-boxes correspond to the odd bits of the 3rd round key. Since the partial subkey values [K<sub>3,2</sub>, K<sub>3,6</sub>, K<sub>3,10</sub>, K<sub>3,14</sub>] has been obtained, for each plaintext/ciphertext sample, we would try all 256 values for the target partial subkey [K<sub>3,1</sub>, K<sub>3,3</sub>, K<sub>3,5</sub>, K<sub>3,7</sub>, K<sub>3,9</sub>, K<sub>3,11</sub>, K<sub>3,13</sub>, K<sub>3,15</sub>]. For each partial subkey value, we would increment the count whenever Equation 7 holds true, where we determine the value of [U<sub>2,1</sub>...U<sub>2,4</sub>], [U<sub>2,5</sub>...U<sub>2,8</sub>] and [U<sub>2,9</sub>...U<sub>2,12</sub>] by running the data backwards through the 3rd and final round of SPN, target partial subkey and S-boxes S21, S22 and S23. Hence, we have simulated attacking our basic cipher by generating 2500 known plaintext/ciphertext values and using linear cryptanalysis, the partial subkey values [K<sub>3,1</sub>, K<sub>3,3</sub>, K<sub>3,5</sub>, K<sub>3,7</sub>, K<sub>3,9</sub>, K<sub>3,11</sub>, K<sub>3,13</sub>, K<sub>3,15</sub>] with the largest bias magnitude is 10111101<sub>2</sub> which corresponded to the target partial subkey value [10111101<sub>2</sub>], confirming that the attack has successfully derived the subkey bits. 

Thus, we managed to recover the 3rd round key = 0x9fa7

# Obtaining the 2nd round key

## Recovering partial subkey values [K<sub>2,2</sub>, K<sub>2,4</sub>, K<sub>2,6</sub>, K<sub>2,8</sub>, K<sub>2,10</sub>, K<sub>2,12</sub>, K<sub>2,14</sub>, K<sub>2,16</sub>]

<img src="https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Trail_2nd_Round_Key_1.png" width="30%">

Since U<sub>1,i</sub>=P<sub>i</sub>+K<sub>1,i</sub>,

> U<sub>1,5</sub> ⨁ U<sub>1,6</sub> ⨁ U<sub>1,7</sub> ⨁ U<sub>1,8</sub> ⨁ U<sub>1,13</sub> ⨁ U<sub>1,14</sub> ⨁ U<sub>1,15</sub> ⨁ U<sub>1,16</sub> = P<sub>5</sub> ⨁ P<sub>6</sub> ⨁ P<sub>7</sub> ⨁ P<sub>8</sub> ⨁ P<sub>13</sub> ⨁ P<sub>14</sub> ⨁ P<sub>15</sub> ⨁ P<sub>16</sub> ⨁ K<sub>1,5</sub> ⨁ K<sub>1,6</sub> ⨁ K<sub>1,7</sub> ⨁ K<sub>1,8</sub> ⨁ K<sub>1,13</sub> ⨁ K<sub>1,14</sub> ⨁ K<sub>1,15</sub> ⨁ K<sub>1,16</sub>

From the above equation:

> U<sub>1,5</sub> ⨁ U<sub>1,6</sub> ⨁ U<sub>1,7</sub> ⨁ U<sub>1,8</sub> ⨁ U<sub>1,13</sub> ⨁ U<sub>1,14</sub> ⨁ U<sub>1,15</sub> ⨁ U<sub>1,16</sub> ⨁ P<sub>5</sub> ⨁ P<sub>6</sub> ⨁ P<sub>7</sub> ⨁ P<sub>8</sub> ⨁ P<sub>13</sub> ⨁ P<sub>14</sub> ⨁ P<sub>15</sub> ⨁ P<sub>16</sub> ⨁ ∑K = 0 (Equation 8)

where ∑K can be 0 or 1 depending on the key of the cipher

As P<sub>i</sub> to U<sub>i</sub> is linear,

> Bias = 0.5

> No of plaintext-ciphertext pairs required >= 1/(1/2)^2 = 4

As the bias is much larger than the bias obtained when recovering the 3rd round key values, the number of plaintext-ciphertext pairs used is much smaller. The linear expression of Equation 8 affects the inputs to S-boxes S<sub>12</sub> and S<sub>14</sub> in the 1st round and the outputs of these S-boxes correspond to the even bits of the 2nd round key. For each partial subkey value, we would increment the count whenever Equation 8 holds true, where we determine the value of [U<sub>1,5</sub>...U<sub>1,8</sub>] and [U<sub>1,13</sub>...U<sub>1,16</sub>] by running the data backwards through the 2nd, 3rd and final round of SPN, target partial subkey and S-boxes S12 and S14. Hence, for each plaintext/ciphertext sample, we would try all 256 values for the target partial subkey [K<sub>2,2</sub>, K<sub>2,4</sub>, K<sub>2,6</sub>, K<sub>2,8</sub>, K<sub>2,10</sub>, K<sub>2,12</sub>, K<sub>2,14</sub>, K<sub>2,16</sub>]. Hence, we have simulated attacking our basic cipher by generating 100 known plaintext/ciphertext values and using linear cryptanalysis, the partial subkey values [K<sub>2,2</sub>, K<sub>2,4</sub>, K<sub>2,6</sub>, K<sub>2,8</sub>, K<sub>2,10</sub>, K<sub>2,12</sub>, K<sub>2,14</sub>, K<sub>2,16</sub>] with the largest bias magnitude is 11001110<sub>2</sub> which corresponded to the target partial subkey value [11001110<sub>2</sub>], confirming that the attack has successfully derived the subkey bits.

## Recovering partial subkey values [K<sub>2,1</sub>, K<sub>2,3</sub>, K<sub>2,5</sub>, K<sub>2,7</sub>, K<sub>2,9</sub>, K<sub>2,11</sub>, K<sub>2,13</sub>, K<sub>2,15</sub>]

<img src="https://github.com/fazli96/CZ4010-Assignment/blob/main/Images/Linear_Trail_2nd_Round_Key_2.png" width="30%">

Since U<sub>1,i</sub>=P<sub>i</sub>+K<sub>1,i</sub>,

> U<sub>1,1</sub> ⨁ U<sub>1,2</sub> ⨁ U<sub>1,3</sub> ⨁ U<sub>1,4</sub> ⨁ U<sub>1,9</sub> ⨁ U<sub>1,10</sub> ⨁ U<sub>1,11</sub> ⨁ U<sub>1,12</sub> = P<sub>1</sub> ⨁ P<sub>2</sub> ⨁ P<sub>3</sub> ⨁ P<sub>4</sub> ⨁ P<sub>9</sub> ⨁ P<sub>10</sub> ⨁ P<sub>11</sub> ⨁ P<sub>12</sub> ⨁ K<sub>1,1</sub> ⨁ K<sub>1,2</sub> ⨁ K<sub>1,3</sub> ⨁ K<sub>1,4</sub> ⨁ K<sub>1,9</sub> ⨁ K<sub>1,10</sub> ⨁ K<sub>1,11</sub> ⨁ K<sub>1,12</sub>

From the above equation:

> U<sub>1,1</sub> ⨁ U<sub>1,2</sub> ⨁ U<sub>1,3</sub> ⨁ U<sub>1,4</sub> ⨁ U<sub>1,9</sub> ⨁ U<sub>1,10</sub> ⨁ U<sub>1,11</sub> ⨁ U<sub>1,12</sub> ⨁ P<sub>1</sub> ⨁ P<sub>2</sub> ⨁ P<sub>3</sub> ⨁ P<sub>4</sub> ⨁ P<sub>9</sub> ⨁ P<sub>10</sub> ⨁ P<sub>11</sub> ⨁ P<sub>12</sub> ⨁ ∑K = 0 (Equation 9)

where ∑K can be 0 or 1 depending on the key of the cipher

As P<sub>i</sub> to U<sub>i</sub> is linear,

> Bias = 0.5

> No of plaintext-ciphertext pairs required >= 1/(1/2)^2 = 4

The linear expression of Equation 9 affects the inputs to S-boxes S<sub>11</sub> and S<sub>13</sub> in the 1st round and the outputs of these S-boxes correspond to the odd bits of the 2nd round key. For each plaintext/ciphertext sample, we would try all 256 values for the target partial subkey [K<sub>2,1</sub>, K<sub>2,3</sub>, K<sub>2,5</sub>, K<sub>2,7</sub>, K<sub>2,9</sub>, K<sub>2,11</sub>, K<sub>2,13</sub>, K<sub>2,15</sub>]. For each partial subkey value, we would increment the count whenever Equation 7 holds true, where we determine the value of [U<sub>1,1</sub>...U<sub>1,4</sub>] and [U<sub>1,9</sub>...U<sub>1,12</sub>] by running the data backwards through the 2nd, 3rd and final round of SPN, target partial subkey and S-boxes S11 and S13. Hence, we have simulated attacking our basic cipher by generating 100 known plaintext/ciphertext values and using linear cryptanalysis, the partial subkey values [K<sub>2,1</sub>, K<sub>2,3</sub>, K<sub>2,5</sub>, K<sub>2,7</sub>, K<sub>2,9</sub>, K<sub>2,11</sub>, K<sub>2,13</sub>, K<sub>2,15</sub>] with the largest bias magnitude is 10011011<sub>2</sub> which corresponded to the target partial subkey value [10011011<sub>2</sub>], confirming that the attack has successfully derived the subkey bits. 

Thus, we managed to recover the 2nd round key = 0xd2de

# Obtaining the 1st round key

Since the 2nd, 3rd, 4th and 5th round key is already obtained, obtaining the 1st round key is to simply take any known plaintext-ciphertext pair, run the ciphertext data backwards through the SPN network until before the 1st key-mixing and XORed the resultant data with its plaintext pair to get the round key.

Thus, we managed to recover the 1st round key = 0x6ff9

# References

* H. M. Heys, “A TUTORIAL ON LINEAR AND DIFFERENTIAL CRYPTANALYSIS”, Cryptologia, vol. 26, no. 3, pp. 189–221, 2002.
* H. M. Heys, "A Tutorial on the Implementation of Block Ciphers: Software and Hardware Applications," Cryptology ePrint Archive, Report 2020/1545, 2020. [Online]. Available: https://ia.cr/2020/1545
