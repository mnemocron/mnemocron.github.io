---
layout: post
title: Sample blog post to learn markdown tips
subtitle: There's lots to learn!
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [test]
comments: true
author: Bill Smith
---

{: .box-note}
This is a demo post to show you how to write blog posts with markdown.  I strongly encourage you to [take 5 minutes to learn how to write in markdown](https://markdowntutorial.com/) - it'll teach you how to transform regular text into bold/italics/tables/etc.<br/>I also encourage you to look at the [code that created this post](https://raw.githubusercontent.com/daattali/beautiful-jekyll/master/_posts/2020-02-28-sample-markdown.md) to learn some more advanced tips about using markdown in Beautiful Jekyll.

# Testing out latex

$$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$


**Here is some bold text**

## Here is a secondary heading

[This is a link to a different site](https://deanattali.com/) and [this is a link to a section inside this page](#local-urls).

Here's a table:

| Number | Next number | Previous number |
| :------ |:--- | :--- |
| Five | Six | Four |
| Ten | Eleven | Nine |
| Seven | Eight | Six |
| Two | Three | One |

How about a yummy crepe?

![Crepe](https://beautifuljekyll.com/assets/img/crepe.jpg)

It can also be centered!

![Crepe](https://beautifuljekyll.com/assets/img/crepe.jpg){: .mx-auto.d-block :}

Here's a code chunk:

~~~
p_latch : process(clk)
begin
  if rising_edge(clk) then
    a_reg <= a;
  end if;
end process p_latch;
~~~

~~~
u32 val,adr;
for(int i=0;i<0x100;i++){
  adr = XPAR_AXI_PCIE_0_BASEADDR + i*4;
  val = Xil_In32(adr);
  xil_printf("%08x %08x\n\r", adr, val);
}
~~~

And here is the same code with syntax highlighting:

```vhdl
p_latch : process(clk)
begin
  if rising_edge(clk) then
    a_reg <= a;
  end if;
end process p_latch;
```

```c
u32 val,adr;
for(int i=0;i<0x100;i++){
  adr = XPAR_AXI_PCIE_0_BASEADDR + i*4;
  val = Xil_In32(adr);
  xil_printf("%08x %08x\n\r", adr, val);
}
```

And here is the same code yet again but with line numbers:

{% highlight vhdl linenos %}
p_latch : process(clk)
begin
  if rising_edge(clk) then
    a_reg <= a;
  end if;
end process p_latch;
{% endhighlight %}

{% highlight c linenos %}
u32 val,adr;
for(int i=0;i<0x100;i++){
  adr = XPAR_AXI_PCIE_0_BASEADDR + i*4;
  val = Xil_In32(adr);
  xil_printf("%08x %08x\n\r", adr, val);
}
{% endhighlight %}

## Boxes
You can add notification, warning and error boxes like this:

### Notification

{: .box-note}
**Note:** This is a notification box.

### Success

{: .box-success}
**Success:** This is a success box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box.

## Local URLs in project sites {#local-urls}

When hosting a *project site* on GitHub Pages (for example, `https://USERNAME.github.io/MyProject`), URLs that begin with `/` and refer to local files may not work correctly due to how the root URL (`/`) is interpreted by GitHub Pages. You can read more about it [in the FAQ](https://beautifuljekyll.com/faq/#links-in-project-page). To demonstrate the issue, the following local image will be broken **if your site is a project site:**

![Crepe](/assets/img/crepe.jpg)

If the above image is broken, then you'll need to follow the instructions [in the FAQ](https://beautifuljekyll.com/faq/#links-in-project-page). Here is proof that it can be fixed:

![Crepe]({{ '/assets/img/crepe.jpg' | relative_url }})
