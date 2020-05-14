# Nock for Everyday Coders, Part 1: Basic Functions and Opcodes
Original Author: `~timluc-miptev`
Revised by: `~digmev-sarvec`

Twitter: [@basile_sportif](https://twitter.com/basile_sportif)

## tldr;/Intro
### For Whom?
Nock is not super complex, and most normal programmers can learn the basics of it rapidly. The mental model you gain from Nock turns out to be **very** useful in learning Hoon and understanding Urbit.

The Urbit documentation generally suggest not to worry about learning Nock. However, the time invested to learn will provide you the insight and basic foundational understanding and as we all like to think, most normal programmers will feel **more** comfortable in Hoon when they know the basics.

By "normal programmer", I mean: a person of somewhat above-average IQ (programming is a pretty self-selecting profession) who works relatively high up the modern tech stack, abstracted from low-level stuff. His/her day job likely consists of a lot of React/JS/SQL/Rails/Java etc, and he/she is not really familiar with assembly/OS stuff.  That describes me and most programmers I know.

What about people who aren't programmers? Anecdotally, they seem to have less trouble "getting" Hoon, so this might not be necessary for them. This might be because people who *have* coded a lot intuitively want to know what is "magic" and what's not in a given language, and learning Nock seems to give a better intuitive feel for that than just starting with Hoon.

### Why Is This Necessary?
The [official Nock docs](https://urbit.org/docs/tutorials/nock/definition/) on the Urbit site are accurate, thorough, and well-explained. However, they take a little while to get to the practical examples, and probably lose a fair number of people along the way. I'm not trying to be original; I'm just a translator.

If I say something here and you're like *"wait, isn't that already in the Nock docs?"*, the answer is **yes, it is; I'm just changing the order for clarity/teaching purposes**, in ways that were helpful to me when I learned.

### Goals by the End of the Series
* To be able to explain Nock clearly in terms most programmers will relate to
* To impart a feeling of confidence with very basic Nock
* To gain a knowledge of Nock’s idioms and big "wins" to support your journey in learning and using Hoon

### Tabel of Contents

<details>
<summary> Nock for Everyday Coders Part 1 </summary>
<ul>
<li> <a href="**Link**">Getting Started and Confusing Points</a> </li>
<li> <a href="**Link**">Nock's Simplest Functions, `0` and `1`</a> </li>
<li> <a href="**Link**">Incrementing Function `4`</a> </li>
<li> <a href="**Link**">Cell-Maker (aka the Distribution Rule)</a> </li>
<li> <a href="**Link**">`3` and `5`, The "Is This a Cell?" and "Equality Test" Functions</a> </li>
<li> <a href="**Link**">The "Subject-Altering" Function</a> </li>
<li> <a href="**Link**">Summary and First Hoon Connections</a> </li>
</ul>
</details>

<details>
<summary> Nock for Everyday Coders Part 2 </summary>
<ul>
<li> <a href="**Link**">An Opening Note about the `1` Function</a> </li>
<li> <a href="**Link**">`6`, "If/Else"</a> </li>
<li> <a href="**Link**">`7`, the "Composition" Opcode</a> </li>
<li> <a href="**Link**">`8`, the "Variable Adder" Opcode</a> </li>
<li> <a href="**Link**">`9`, Create a Core and Run One of Its Arms</a> </li>
<li> <a href="**Link**">Real Nock Code</a> </li>
</ul>
</details>
