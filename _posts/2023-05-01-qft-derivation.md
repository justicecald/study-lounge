---
layout: post
title: A Derivation of the Quantum Fourier Transform
author: Justice M. Calderón
tags: [software engineering, computer science, mathematics, quantum computing]
---

Hey Quantum Computing enthusiast! My name is Justice Calderón, I'm a mechanical engineer turned software engineer, and in this post I'm going to walkthrough my version of a derivation of the Quantum Fourier Transform. 

Self-learning quantum computing can be difficult. But taking time to learn a practice implementing the mathematical principles can be beneficial to starting a career in Quantum Information Science.

Without any further ado, let's jump in!

# The derivation - Some algebra

The Quantum Fourier Transform (or QFT for short) is a vital sub-module to some of the most powerful quantum algorithms today. QFT is used in Quantum Phase Estimation (QPE), a process that is used in determining the eigenvalue of an arbitrary unitary operator, $$\mathcal U $$, and in the famous [Shor's Algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm). 

If you're interested in following along or making your own notes on my derivation, I used the following texts for my work:
1. [Quantum Computation and Quantum Information by Mike & Ike](https://www.amazon.com/Quantum-Computation-Information-10th-Anniversary/dp/1107002176/ref=sr_1_1?keywords=quantum+computation+and+quantum+information&qid=1682973795&sprefix=quantum+compu%2Caps%2C181&sr=8-1&ufe=app_do%3Aamzn1.fos.006c50ae-5d4c-4777-9bc0-4513d670b6bc)
2. [An Introduction to Quantum Computing by Kaye, LaFlamme, and Mosca](https://www.amazon.com/Introduction-Quantum-Computing-Phillip-Kaye/dp/019857049X/ref=sr_1_4?crid=LHSVWJTMAQIK&keywords=an+introduction+to+quantum+computing&qid=1682973871&sprefix=an+introduction+to+quantum+computin%2Caps%2C120&sr=8-4&ufe=app_do%3Aamzn1.fos.006c50ae-5d4c-4777-9bc0-4513d670b6bc)
3. [Problems and Solutions in Quantum Computing and Quantum Information by Steeb and Hardy](https://www.amazon.com/Problems-Solutions-Quantum-Computing-Information/dp/981323928X/ref=sr_1_1?crid=2V3WIZLJ0MIN5&keywords=problems+and+solutions+in+quantum+computing&qid=1682973925&sprefix=problems+and+solutions+in+quantum+computing%2Caps%2C113&sr=8-1)
4. [Mathematics of Quantum Computing, an Introduction by Scherer](https://www.amazon.com/Mathematics-Quantum-Computing-Wolfgang-Scherer/dp/303012360X/ref=sr_1_4?crid=29NEAGDQSEGW3&keywords=mathematics+of+quantum+computing&qid=1682973973&sprefix=mathematics+of+quantum+computing%2Caps%2C120&sr=8-4&ufe=app_do%3Aamzn1.fos.006c50ae-5d4c-4777-9bc0-4513d670b6bc)

Now, QFT deals with representing (or transforming) a state $$\ket{x}$$ into it's "Fourier form", $$\sum_k{\ket{k}}$$ with some function $$f(x)$$. We can write this process out explicitly as:

$$
\ket{x} \xrightarrow{QFT} \frac{1}{2^{n/2}} \sum_{k=0}^{2^{n}-1} \text{exp}\left [ i2\pi \left (\frac{kx}{2^{n}} \right ) \right ]\ket{k}
$$

Where we can show the QFT operator, $$ \hat{\mathcal{F}}$$ as:

$$
\hat{\mathcal{F}} \equiv \frac{1}{2^{n/2}} \sum_{k=0}^{2^{n}-1} \text{exp}\left [ i2\pi \left (\frac{kx}{2^{n}} \right ) \right ]\ket{k}\bra{x}
$$

For $$x,k \in \{0,1\}^{n}$$. To get an idea of how we'd implement this as an algorithm on a quantum computer, let's express this protocol operating on an arbitrary state, $$\ket{x}$$.

We'll start by stating that the states $$\ket{x}$$ and $$\ket{k}$$ will be expressed by their _**binary strings**_ of length $$n$$ i.e., 

$$
\ket{x} = \ket{x_1 x_2 ... x_n}
$$

and 

$$
x = \sum_{l=0}^{n-1}x_l 2^l
$$

Now, stepping through operating $$\mathcal{F}$$ on $$\ket{x}$$:

$$
\hat{\mathcal{F}}\ket{x} = \left ( \frac{1}{2^{n/2}} \sum_{k=0}^{2^{n}-1} \text{exp}\left [ i2\pi \left (\frac{kx}{2^{n}} \right ) \right ]\ket{k}\bra{x} \right ) \ket{x}
$$

The algebra on the RHS goes as:

$$
\frac{1}{2^{n/2}} \sum_{k=0}^{2^{n}-1} \text{exp}\left [ i2\pi \left (\frac{kx}{2^{n}} \right ) \right ]\ket{k}
$$

$$
\frac{1}{2^{n/2}} \sum_{k_0 = 0}^{1} ... \sum_{k_l = 0}^{1} \text{exp}\left [ i2\pi \left (\frac{kx}{2^{n}} \right ) \right ]\ket{k_0 k_1 ... k_{n-1}}
$$

$$
\frac{1}{2^{n/2}} \sum_{k_0 = 0}^{1} ... \sum_{k_l = 0}^{1} \text{exp}\left [ i2\pi \left (\frac{x}{2^{n}} \right ) \sum_{l=0}^{n-1}k_l 2^l\right ]\ket{k_0 k_1 ... k_{n-1}}
$$

So far, in lines 1-2 of the RHS algebra, we've plugged in the binary string notation and including a sum over all possible values of $$k_l$$ which is $$0$$ or $$1$$ (the only measurable values of the qubit, or eigenvectors). In lines 2-3, we plugged in the binary sum notation for computing the value that the binary string $$k_0k_1...k_{n-1}$$ is representing.

If you're unfamiliar with this binary sum idea, think of it as the way we compute numbers in decimal. For example, the number 321 in decimal is represented by:

$$
x_0x_1x_2 = 3(10^2) + 2(10^1)  + 1(10^0)
$$

The binary sum works _**the same way**_ just with a base of $$2$$ instead of $$10$$. Now, moving on, we can take advantage of the fact that there is a summation in the exponential term, and expand this into a product.

$$
\frac{1}{2^{n/2}} \prod_{l=0}^{n-1}\text{exp}\left [ i2\pi x k_l 2^{l-n}\right ]\sum_{k_0, ..., k_{n-1} = 0}^{1}\ket{k_0 k_1 ... k_{n-1}}
$$

Next, we're going to use _**tensor product**_ notation to compact the bit string $$\ket{k_0 k_1 ... k_{n-1}}$$. That is:

$$
\ket{k_0 k_1 ... k_{n-1}} = \bigotimes_{l=0}^{n-1} \ket{k_l}
$$

If this notation makes you a bit nervous, don't be. It's just another operator that we can perform in linear algebra. If you'd like to learn more about tensor products, I'd recommend visiting this [link](https://en.wikipedia.org/wiki/Tensor_product) or referring to [Mathematics of Quantum Computing, an Introduction](https://www.amazon.com/Mathematics-Quantum-Computing-Wolfgang-Scherer/dp/303012360X/ref=sr_1_4?crid=29NEAGDQSEGW3&keywords=mathematics+of+quantum+computing&qid=1682973973&sprefix=mathematics+of+quantum+computing%2Caps%2C120&sr=8-4&ufe=app_do%3Aamzn1.fos.006c50ae-5d4c-4777-9bc0-4513d670b6bc). 

Next, plugging this in, we get the following for the RHS:

$$
\frac{1}{2^{n/2}} \prod_{l=0}^{n-1}\text{exp}\left [ i2\pi x k_l 2^{l-n}\right ]\bigotimes_{l=0}^{n-1}\sum_{k_l = 0}^{1}\ket{k_l}
$$

$$
\frac{1}{2^{n/2}} \bigotimes_{l=0}^{n-1}\sum_{k_l = 0}^{1}e^{i2\pi x k_l 2^{l-n}}\ket{k_l}
$$

Where in the second line, we simply stuffed the exponential term into the tensor product since it would give us the result we were looking for while associating each exponent with the correct ket.

# The derivation - the quantum circuit

$$
\hat{\mathcal{F}}\ket{x}=\frac{1}{2^{n/2}} \bigotimes_{l=0}^{n-1}\sum_{k_l = 0}^{1}e^{i 2 \pi x k_l 2^{l-n}}\ket{k_l}
$$

Awesome! Now that we have an expression for the result, let's uncover how we should act this on the qubits a part of state $$\ket{x}$$. First, we'll start by using the binary representation of $$\ket{x}$$ in our expression:

$$
\hat{\mathcal{F}}\ket{x}=\frac{1}{2^{n/2}}\bigotimes_{l=0}^{n-1}\sum_{k_l = 0}^{1}e^{i 2 \pi \left (\sum_{j=0}^{n-1}x_j 2^j \right ) k_l 2^{l-n}}\ket{k_l}
$$

$$
\hat{\mathcal{F}}\ket{x}=\frac{1}{2^{n/2}}\bigotimes_{l=0}^{n-1}\left (\ket{0} + e^{i 2 \pi \left (\sum_{j=0}^{n-1}x_j 2^{l-n+j} \right )}\ket{1} \right )
$$

Where the only "trick" we did really was to expand the sum over the k kets into the states we know they will take, i.e., $$\ket{0}$$ and $$\ket{1}$$. Now, let's look into the exponential term tied with $$\ket{1}$$ a bit closer...

If we were to split the exponential term into **two** terms for when $$(l-n+j) \geq 0$$ and $$(l-n+j) < 0$$, we get:

$$
\text{exp}{\left [i 2 \pi \sum_{j=0}^{n-1}x_j 2^{l-n+j} \right ]} = \text{exp}{\left [i 2 \pi \sum_{j=-l+n}^{n-1}x_j 2^{l-n+j} \right ]} \times \text{exp}{\left [i 2 \pi \sum_{j=0}^{-l+n-1}x_j 2^{l-n+j} \right ]}
$$

Now we can see that the complex exponential in the first term on the RHS of the above equation takes the form of $$e^{ic2\pi}$$ where $$c \in \mathbb{Z}$$. Therefore, the first term only amounts to complete rotations of $$2\pi$$ leaving the term equal to one! All that's left is the second term:

$$
\hat{\mathcal{F}}\ket{x}=\frac{1}{2^{n/2}}\bigotimes_{l=0}^{n-1}\left (\ket{0} + e^{i 2 \pi \left (\sum_{j=0}^{n-l-1}x_j 2^{l-n+j} \right )}\ket{1} \right )
$$

Finally, we get to clean up a bit of this mess! Let's make note that the term in the parenthesis is a _superposition_ state of a **single qubit**. Namely, the $$l^{\text{th}}$$ qubit. So, for $$l=n-1$$, the exponential term with the $$2$$ as the base goes from $$2^{l-n+j} \rightarrow 2^{j-n}$$. Thus, the term can only range from $$2^{-n}$$ to $$2^{-1}$$. Then looking at the summation, this means that for the last qubit ($$l=n-1$$) there is **one** term in the summation, while for the $$0^{\text{th}}$$ qubit, there are $$n-1$$ terms. 

If we then leverage the definition of [_**binary fractions**_](https://www.electronics-tutorials.ws/binary/binary-fractions.html), i.e., 

$$
0.x_l...x_{n-1} = \sum_{j=l}^{n-1} x_j 2^{-j}
$$

Then we can conclude that the summation in $$\ket{1}$$'s phase term is just the binary fraction expansion of the bit string $$x_l...x_{n-1}$$! If you're getting confused with notation, just note that the $$-j$$ is communicating that $$j<0$$ and not that a minus sign is missing from our expression. This we can write out as:

$$
\hat{\mathcal{F}}\ket{x} = \frac{1}{2^{n/2}} \bigotimes_{l=0}^{n-1}\left ( \ket{0} + e^{i 2 \pi0.x_l...x_{n-1}}\ket{1}\right)
$$

As a small toy example, if we wanted to compute the QFT of $$\ket{0}$$, then we would write this out as:

$$
\hat{\mathcal{F}}\ket{0} = \frac{1}{2^{1/2}}(\ket{0} + e^{i2\pi0.x}\ket{1})
$$

$$
\hat{\mathcal{F}}\ket{0} = \frac{1}{2^{1/2}}(\ket{0} + e^{i2\pi(0.0)}\ket{1})
$$

$$
\hat{\mathcal{F}}\ket{0} = \frac{1}{2^{1/2}}(\ket{0} + \ket{1}) = \hat{\mathcal{H}}\ket{0}
$$

Pretty neat, huh!

# Conclusion

![QFT Circuit](https://github.com/justicecald/study-lounge/tree/master/images/qft-circuit.jpeg)
_Quantum Circuit from Quantum Computation and Quantum Information pg. 219_

As you grow the number of qubits in your QFT computation (i.e., increasing $$n$$), you'll have more terms to be concerned with. Don't worry! Even thought the operations won't be as simple as applying a Hadamard gate, the implementation is still pretty straight forward. For $$n>1$$, the name of the game will be to apply controlled rotations or CNOT gates conditioned on other qubits in your computation. Not too bad!

I hope you enjoyed this derivation! This was pretty fun and interesting to me. I look forward to doing more as time goes on!

Signing off!
