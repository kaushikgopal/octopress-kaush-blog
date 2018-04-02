---
layout: post
title: Smarter ToDos with Kotlin
date: 2018-03-14T08:30:00-08:00
external-url: https://tech.instacart.com/smarter-todos-with-kotlin-beb522fe9a01
---

Checkout this quick blog post I wrote for my company, tweaking the existing Kotlin TODO to work towards our requirements.

While I don't think this solution is a panacea for all your missing code snippets, I have found some luck with this method, in adding accountability for those PR review feedback comments you say you'll get to, but conveniently forget :)

Here's a bonus if you're reading this article from here:

{% codeblock lang:kotlin linenos:false %}
/**
 * @param month - regular month (so 3 = March)
 */
@JvmOverloads
fun ISDate(
    day: Int? = null,
    month: Int? = null,
    year: Int? = null,
    hour: Int? = null,
    minute: Int? = null,
    second: Int? = null,
    date: Date? = null
): Date = Date().apply {

    val cal = Calendar.getInstance()

    date?.let { cal.time = it }
    day?.let { cal.set(Calendar.DAY_OF_MONTH, it) }
    month?.let { cal.set(Calendar.MONTH, it - 1) }
    year?.let { cal.set(Calendar.YEAR, it) }
    hour?.let { cal.set(Calendar.HOUR_OF_DAY, it) }
    minute?.let { cal.set(Calendar.MINUTE, it) }
    second?.let { cal.set(Calendar.SECOND, it) }

    return cal.time
}
{% endcodeblock %}

It’s a pretty straightforward date builder with minor tweaks (for e.g. 3 = March cause what are we, Monsters ?).

