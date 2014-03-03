---
layout: post
title:  "Converting Bytes to Hex Strings In Java"
date:   2014-02-08
---

<p>There seems to be a lot of misinformation and overly complicated solutions to the problem of converting byte values (and arrays) to hex strings in Java floating around on the internet. In this post I am to clear up the confusion and provide a clean solution to the problem. To illustrate I will provide a fairly straightforward approach, but I'll provide a bit more detail to help illustrate <em>why</em> it works.</p>
<p>First off, what are we trying to do? We want to convert a byte value (or an array of bytes) to a string which represents a hexadecimal value in ASCII. So step one is to find out exactly what a byte in Java is:</p>
<blockquote><p>The byte data type is an <strong>8-bit signed two's complement integer</strong>. It has a minimum value of -128 and a maximum value of 127 (inclusive). The byte data type can be useful for saving memory in large arrays, where the memory savings actually matters. They can also be used in place of int where their limits help to clarify your code; the fact that a variable's range is limited can serve as a form of documentation.</p>
</blockquote>
<p>What does this mean? A few things: First and most importantly, it means we are working with <strong>8-bits</strong>. So for example we can write the number 2 as 0000 0010. However, since it is two's complement, we write a negative 2 like this: 1111 1110. What is also means is that converting to hex is very straightforward. That is, you simply convert each 4 bit segment directly to hex. Note that to make sense of negative numbers in this scheme you will first need to understand two's complement. If you don't already understand two's complement, you can read an excellent explanation, here: <a href="http://www.cs.cornell.edu/~tomf/notes/cps104/twoscomp.html">http://www.cs.cornell.edu/~tomf/notes/cps104/twoscomp.html</a>     </p>
<p>&#160;</p>
<h1>Converting Two's Complement to Hex In General </h1>
<p>Once a number is in two's complement it is dead simple to convert it to hex. In general, converting from binary to hex is very straightforward, and as you will see in the next two examples, you can go directly from two's complement to hex. </p>
<h2>Examples </h2>
<p><strong>Example 1:</strong> Convert 2 to Hex. </p>
<blockquote><p>1) First convert 2 to binary in two's complement: </p>
<p>&#160;&#160;&#160; 2 (base 10) = 0000 0010 (base 2)</p>
<p>2) Now convert binary to hex:</p>
<p>&#160;&#160;&#160; 0000 = 0x0 in hex      <br />&#160;&#160;&#160; 0010 = 0x2 in hex</p>
<p>&#160;&#160;&#160; therefore 2 = 0000 0010 = 0x02. </p>
</blockquote>
<p><strong>Example 2:</strong> Convert -2 (in two's complement) to Hex.</p>
<blockquote><p>1) First convert -2 to binary in two's complement: </p>
<p>&#160;&#160;&#160; -2 (base 10) = 0000 0010 (direct conversion to binary)      <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 1111 1101 (invert bits)       <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; 1111 1110 (add 1)       <br />&#160;&#160;&#160; therefore: -2 = 1111 1110 (in two's complement)</p>
<p>2) Now Convert to Hex:</p>
<p>&#160;&#160;&#160; 1111 = 0xF in hex      <br />&#160;&#160;&#160; 1110 = 0xE in hex</p>
<p>&#160;&#160;&#160; therefore: -2 = 1111 1110 = 0xFE.</p>
</blockquote>
<p>&#160;</p>
<h1>Doing this In Java</h1>
<p>Now that we've covered the concept, you'll find we can achieve what we want with some simple masking and shifting. The key thing to understand is that <strong>the byte you are trying to convert is already in two's complement.</strong> You don't do this conversion yourself. I think this is a major point of confusion on this issue. Take for example the follow byte array:</p>
<blockquote><p>&#160;&#160;&#160; byte[] bytes = new byte[]{-2,2};</p>
</blockquote>
<p>We just manually converted them to hex, above, but how can we do it in Java? Here's how: </p>
<p><strong>Step 1:</strong> Create a StringBuffer to hold our computation.&#160; </p>
<blockquote><p>&#160;&#160;&#160; StringBuffer buffer = new StringBuffer();</p>
</blockquote>
<p><strong>Step 2:</strong> Isolate the higher order bits, convert them to hex, and append them to the buffer</p>
<p>Given the binary number 1111 1110, we can isolate the higher order bits by first shifting them over by 4, and then zeroing out the rest of the number. Logically this is simple, however, the implementation details in Java (and many languages) introduce a wrinkle because of sign extension. Essentially, when you shift a byte value, Java first converts your value to an integer, and then performs sign extension. So while you would expect 1111 1110 &gt;&gt; 4 to be 0000 1111, in reality, in Java it is represented as the two's complement 0xFFFFFFFF!</p>
<p>So returning to our example: </p>
<blockquote><p>&#160;&#160;&#160; 1111 1110 &gt;&gt; 4 (shift right 4) = 1111 1111 1111 1111 1111 1111 1111 1111 (32 bit sign-extended number in two's complement)</p>
</blockquote>
<p>We can then isolate the bits with a mask:</p>
<blockquote><p>&#160;&#160;&#160; 1111 1111 1111 1111 1111 1111 1111 1111 &amp; 0xF = 0000 0000 0000 0000 0000 0000 0000 1111      <br />&#160;&#160;&#160; therefore: 1111 = 0xF in hex. </p>
</blockquote>
<p>In Java we can do this all in one shot:    <br />&#160; </p>
<blockquote><p>&#160;&#160;&#160; Character.forDigit((bytes[0] &gt;&gt; 4) &amp; 0xF, 16);</p>
</blockquote>
<p>The forDigit function just maps the number you pass it onto the set of hexadecimal numbers 0-F. </p>
<p><strong>Step 3: </strong>Next we need to isolate the lower order bits. Since the bits we want are already in the correct position, we can just mask them out:</p>
<blockquote><p>&#160;&#160;&#160; 1111 1110 &amp; 0xF = 0000 0000 0000 0000 0000 0000 0000 1110 (recall sign extension from before)      <br />&#160;&#160;&#160; therefore: 1110 = 0xE in hex.&#160; </p>
</blockquote>
<p>Like before, in Java we can do this all in one shot:</p>
<blockquote><p>&#160;&#160;&#160; Character.forDigit((bytes[0] &amp; 0xF), 16);</p>
</blockquote>
<p>Putting this all together we can do it as a for loop and convert the entire array:</p>
<blockquote><p>&#160;&#160;&#160; for(int i=0; i &lt; bytes.length; i++){      <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160; buffer.append(Character.forDigit((bytes[i] &gt;&gt; 4) &amp; 0xF, 16));       <br />&#160;&#160;&#160;&#160;&#160;&#160;&#160; buffer.append(Character.forDigit((bytes[i] &amp; 0xF), 16));       <br />&#160;&#160;&#160; }</p>
</blockquote>
<p>Hopefully this explanation makes things clearer for those of you wondering exactly what is going on in the many examples you will find on the internet. Hopefully I didn't make any egregious errors, but suggestions and corrections are highly welcome!&#160; </p>
