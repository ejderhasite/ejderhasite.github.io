---
layout: post
title: On More Prisoners
date: 2017-07-02 11:30
comments: true
external-url:
categories: Mathematics
custom_js:
- prisoners
---

> 100 prisoners are imprisoned in solitary cells. Each cell is windowless and soundproof. There's a central living room with one light bulb; the bulb is initially off. No prisoner can see the light bulb from his or her own cell. Each day, the warden picks a prisoner equally at random, and that prisoner visits the central living room; at the end of the day the prisoner is returned to his cell. While in the living room, the prisoner can toggle the bulb if he or she wishes. Also, the prisoner has the option of asserting the claim that all 100 prisoners have been to the living room. If this assertion is false (that is, some prisoners still haven't been to the living room), all 100 prisoners will be shot for their stupidity. However, if it is indeed true, all prisoners are set free and inducted into MENSA, since the world can always use more smart people. Thus, the assertion should only be made if the prisoner is 100% certain of its validity.

## First approach

Let $C$ be a random variable that denotes the counter being called, and $NC$ a non-counter. Then:

$$
\begin{eqnarray*}
 C &\sim& NBin(1, \frac{1}{100}) \\
 NC(x) &\sim& NBin(1, \frac{x}{100})
\end{eqnarray*}
$$

... where $x$ is the number of prisoners that are yet to be accounted for; this step is important, since every time a prisoner turns on a switch, the probability of picking one that is yet to perform such action decreases.

Finally, we can express the expected value as:

$$\sum_{i=1}^{99} { \mathbb{E}[C] + \mathbb{E}[NC(100-i)]}$$

## Let's remember some useful stuff

Recall the linearity of expectation; a negative binomial can be expanded as something like this:

case	| calls | probability |
------|-------|-------------|
p     | 1     | $p$         |
qp    | 2     | $pq$        |
qqp   | 3     | $pq^2$      |
...   | ...   | ...         |
...   | n     | $pq^{n-1}$  |

The mean (or expected value) is, by definition, the sum of all possible values multiplied by their respective probabilities:

$$
\begin{equation*}
\mu_X = \mathbb{E}[X] = \sum_{x\in X}x\cdot p(x)
\end{equation*}
$$

Hence, considering $q = (1-p)$:

$$1p + 2pq + 3pq^2 + \cdots + npq^{n-1} = \sum_{i=1}^{\infty} np(1-p)^{n-1} = \frac{1}{p}$$

It is easy to that the infinite sum converges, and we get a closed form probability function. And since $p=\frac{1}{100}$, then $\mathbb{E}[C] = \class{bghighlight}{100}$. The conclusion is actually very intuitive: on average, one has to call 100 prisioners before our *counter* is selected.

## I heard something about "distributions" ...

Alternatively, we also know that the expected value of a random variable following a [negative binomial distribution](https://en.wikipedia.org/wiki/Negative_binomial_distribution), up to and including the success event happening ($r = 1$, where $r$ is the number of successes that need to happen), is:

$$\mathbb{E}[C] = r\frac{1-p}{p} +1= 1\frac{1-\frac{1}{100}}{\frac{1}{100}} + 1 = \class{bghighlight}{100}$$

Similarly, the expected value for a prisioner that has never turned on the switch, and considering that the probability depends on how many prisioners still have that property[^2]:

[^2]: Expressing the same using linearity of expectation is left as an exercise to the reader.

$$\mathbb{E}[NC(x)] = 1\frac{1-\frac{x}{100}}{\frac{x}{100}} + 1 = \frac{100}{x}$$

...where $$x = \{99, 98, \ldots, 1\}$$. We now have all the elements to calculate the expected value, so:

$\phantom{\equiv}~\sum_{i=1}^{99} \mathbb{E}[C] + \mathbb{E}[NC(100-i)]$ <br>
$\equiv$ { since $\Sigma (a+b) = \Sigma a+\Sigma b$ }<br>
$\phantom{\equiv}~\sum_{i=1}^{99} \mathbb{E}[C] + \sum_{i=1}^{99} \mathbb{E}[NC(100-i)]$ <br>
$\equiv$ { solving left side } <br>
$\phantom{\equiv}~9900 + \sum_{i=1}^{99} \mathbb{E}[NC(100-i)]$ <br>
$\equiv$ { by definition } <br>
$\phantom{\equiv}~9900 + \sum_{i=1}^{99} \frac{100}{100-i}$ <br>
$\equiv$ { solving right side } <br>
$\phantom{\equiv}~9900 + 517.74 = \class{bghighlight}{10417.74}$

## Simulation

I like to solve the same problem using more than one technique; so, as always, we shall simulate it:

```javascript
const random = (min, max) => Math.floor(Math.random() * (Math.floor(max) - Math.ceil(min) + 1)) + Math.ceil(min)
const avg = (r) => r.reduce((a, b) => a+b) / r.length

let n = 100
let counterGuy = random(1, 100)

let results = [...Array(n).keys()].map(() => {
  let counter = 0
  let sw = false
  let steps = 0
  let mem = new Set()

  while(counter < 99) {
    steps++
    var p = random(1, 100)
    if (p == counterGuy) {
      if (sw == true) {
        counter++
        sw = false
      }
    } else {
      if (sw == false && !mem.has(p)) {
        sw = true
        mem.add(p)
      }
    }
  }

  return steps;
})

console.log(`After ${n} simulations, the expected value is ${avg(results)}`)
```

## Fancy chart

Of course, the Expected Value is exactly what the name says: expected. On average, it will be 10417.74, but in reality it is a random variable that follows a normal distribution, centered at that value. We can run the above simulation, say, 5000 times, and then plot an histogram to get the feeling of it, like so:

<div id="histogram"></div>