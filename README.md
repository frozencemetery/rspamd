[![CircleCI](https://circleci.com/gh/vstakhov/rspamd/tree/master.svg?style=svg)](https://circleci.com/gh/vstakhov/rspamd/tree/master)

## Introduction

[Rspamd](https://rspamd.com) is an advanced spam filtering system that allows evaluation of messages by a number of
rules including regular expressions, statistical analysis and custom services
such as URL black lists. Each message is analysed by rspamd and given a `spam score`. 

According to this spam score and the user's settings rspamd recommends an action for
the MTA to apply to the message- for example to pass, reject or add a header.
Rspamd is designed to process hundreds of messages per second simultaneously and has a number of 
features available.

You can watch the following [introduction video](https://www.youtube.com/watch?v=_fl9i-az_Q0) from the [FOSDEM-2016](http://fosdem.org) where I describe the main features of rspamd and explain why rspamd runs so fast.

Rspamd is [packaged](https://rspamd.com/downloads.html) for the major Linux distributives and is also available via [FreeBSD ports](https://freshports.org/mail/rspamd) and NetBSD [pkgsrc](https://pkgsrc.org).

## Spam filtering features

Rspamd distribution contains a number of mail processing features, including such techniques as:

* **Regular expressions filtering** - allows basic processing of messages, their textual parts, MIME headers and
SMTP data received by MTA against a set of expressions that includes both normal regular expressions and 
message processing functions. Rspamd expressions are the powerful tool that allows to filter messages based on
some pre-defined rules. This feature is similar to regular expressions in spamassassin spam filter.


* **SPF module** that allows to validate a message's sender against the policy defined in the DNS record of sender's domain. You can read
about SPF policies [here](http://www.openspf.org/). A number of mail systems includes SPF support, such as `gmail` or `yahoo mail`.


* **DKIM module** validates a message cryptographic signature against a public key placed in the DNS record of sender's domain. Like SPF,
this technique is widely spread and allows to validate that a message is sent from that specific domain.


* **DNS black lists** allows to estimate reputation of sender's IP address or network. Rspamd uses a number of DNS lists including such lists as
`SORBS` or `spamhaus`. However, rspamd doesn't trust any specific DNS list and use a conjunction of estimations instead that allows to
avoid mistakes and false positives. Rspamd also uses positive and grey DNS lists for checking for trusted senders.


* **URL black lists** are rather similar to DNS black lists but uses URLs in a message to make an estimation of sender's reputation.
This technique is very useful for finding malicious or phished domains and filter such mail.


* **Statistics** - rspamd uses bayesian classifier based on five-gramms of input. This means that the input is estimated not based on individual
words, but all input is organized in chains that are further estimated by bayesian classifier. This approach allows to achieve better results than
traditionally used monogramms (or words literally speaking), that is described in details in the following [paper](http://osbf-lua.luaforge.net/papers/osbf-eddc.pdf).


* **Fuzzy hashes** - for checking of malicious mail patterns rspamd uses so called `fuzzy hashes`. Unlike normal hashes, these structures are targeted to hide
small differences between text patterns allowing to find similar messages quickly. Rspamd has internal storage of such hashes and allows to block mass spam sendings
quickly based on user's feedback that specifies messages reputation. Moreover, this allows to feed rspamd with data from [`honeypots`](http://en.wikipedia.org/wiki/Honeypot_(computing)#Spam_versions)
without polluting the statistical module.

Rspamd uses the conjunction of different techniques to make the finall decision about a message. This allows to improve the overall quality of filtering and reduce the number of
false positives (e.g. when a innocent message is badly classified as a spam one). I have tried to simplify rspamd usage by adding the following elements:

* **Web interface** - rspamd is shipped with the fully functional ajax-based web interface that allows to observe rspamd statistic, to configure rules, weights and lists, to scan
and learn messages and to view the history of scans. The interface is self-hosted, requires zero configuration and follows the recent web applications standards. You don't need a
web server or applications server to run web UI - you just need to run rspamd itself and a web browser.

* **Integration with MTA** - rspamd can work with the most popular mail transfer systems, such as postfix, exim or sendmail. For postfix and sendmail, there is an [`rmilter` project](https://github.com/vstakhov/rmilter),
whilst for exim there are several solutions to work with rspamd. Should you require MTA integration then please consult with the [integration guide](https://rspamd.com/doc/integration.html).

* **Extensive LUA API** - rspamd ships with hundreds of [lua functions](https://rspamd.com/doc/lua) that are available to write own rules for efficient and targeted spam filtering.

* **Dynamic tables** - rspamd allows to specify bulk lists as `dynamic maps` that are checked in runtime with updating data when they are changed. Rspamd supports file, HTTP and HTTPS maps.

## Performance

Rspamd is designed to be fast. The core of rspamd is written in `C` and uses an event-driven model that allows to process multiple messages simultaneously and without blocking.
Moreover, a set of techniques is used in rspamd to process messages faster:

* **Finite state machines processing** - rspamd uses specialized finite state machines for the performance critical tasks to process input faster than a set of regular expressions.
Of course, it is possible to implement these machines by ordinary `perl regular expressions` but then they won't be compact or human-readable. On the contrary, rspamd optimizes
such actions as headers processing, received elements extraction, protocol operations by building the concrete automata for an assigned task.

* **Expressions optimizer** - allows to optimize expressions by execution of `likely false` or `likely true` expressions in order in the branches. That allows to reduce number of
expensive expressions calls when scanning a message.

* **Symbols optimizer** - rspamd tries to check first the rules that are frequent or inexpensive in terms of time or CPU resources which allows to block spam before processing of
expensive rules (rules with negative weights are always checked before other ones).

* **Event driven model** - rspamd is designed not to block anywhere in the code and knowing that a spam check requires a lot of network operations, rspamd can process many messages
simultaneously increasing the efficiency of shared DNS caches and other system resources. Moreover, event-driven system normally scales automatically and you won't need to do any
tuning in the most of cases.

* **Hyperscan regular expressions engine** - rspamd utilizes [hyperscan](https://01.org/hyperscan) engine to match multiple regular expressions at the same time. You can watch the following [presentation](https://highsecure.ru/rspamd-hyperscan.pdf) where the main benefits of hyperscan are described.

* **Clever choice of data structures** - rspamd tries to use the optimal data structure for each task. For example it uses very efficient suffix tries for fast matching of a text
against a set of multiple patterns. Or it uses radix bit trie for storing IP addresses information that provides O(1) access time complexity.

## Extensions

Besides its `C` core, rspamd provides an extensive [Lua](http://lua.org) API to access almost all the features available directly from `C`. Lua is an extremely easy
to learn programming language though it is powerful enough to implement complex mail filters. In fact rspamd has a significant amount of code written completely in Lua such as
DNS blacklists checks, user's settings or different maps implementation. You can also write your own filters and rules in Lua adopting rspamd functionality to your needs.
Furthermore, Lua programs are very fast and their performance is rather [close](http://attractivechaos.github.io/plb/) to pure `C`. However, you should mention that for the most
of performance critical tasks you usually use the rspamd core functionality than Lua code. Anyway, you can also use `LuaJIT` with rspamd if your goal is maximum performance.
From the Lua API you can do the following tasks:

* **Reading the configuration parameters** - Lua code has the full access to the parsed configuration knobs and you can easily modify your plugins behaviour by means of the main
rspamd configuration

* **Registering custom filters** - it is more than simple to add your own filters to rspamd: just add new index to the global variable `rspamd_config`:

~~~lua
rspamd_config.MYFILTER = function(task)
-- Do something
end
~~~

* **Full access to the content of messages** - you can access text parts, headers, SMTP data and so on and so forth by using of `task` object. The full list of methods could be found
[here](https://rspamd.com/doc/lua/task.html).


* **Pre- and post- filters** - you can register callbacks that are called before or after messages processing to make results more precise or to make some early decision,
for example to implement a rate limit.

* **Registering functions for rspamd** - you can write your own functions in Lua to extend rspamd internal expression functions.

* **Managing statistics** - Lua scripts can define a set of statistical files to be scanned or learned for a specific message allowing to create more complex
statistical systems, e.g. based on an input language. Moreover, you can even learn rspamd statistic from Lua scripts.

* **Standalone Lua applications** - you can even write your own worker based on rspamd core and performing some asynchronous logic in Lua. Of course, you can use the
all features from rspamd core, including such features as non-blocking IO, HTTP client and server, non-blocking redis client, asynchronous DNS, UCL configuration and so on
and so forth.

* **API documentation** - rspamd Lua API has an [extensive documentation](https://rspamd.com/doc/lua) where you can find examples, references and the guide about how to extend
rspamd with Lua.


## References

* Home site: <https://rspamd.com>
* Development: <https://github.com/vstakhov/rspamd>
