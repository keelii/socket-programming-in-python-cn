## 背景知识

Socket 有一段很长的历史，最初是在 [1971 年被用于 ARPANET](https://en.wikipedia.org/wiki/Network_socket#History)，随后就成了 1983 年发布的 Berkeley Software Distribution (BSD) 操作系统的 API，并且被命名为 [Berkeley socket](https://en.wikipedia.org/wiki/Berkeley_sockets)

当互联网在 20 世纪 90 年代随万维网兴起时，网络编程也火了起来。Web 服务和浏览器并不是唯一使用新的连接网络和 Socket 的应用程序。各种类型不同规模的客户端 / 服务器应用都广泛地使用着它们

时至今日，尽管 Socket API 使用的底层协议已经进化了很多年，也出现了许多新的协议，但是底层的 API 仍然保持不变

Socket 应用最常见的类型就是 **客户端 / 服务器** 应用，服务器用来等待客户端的链接。我们教程中涉及到的就是这类应用。更明确地说，我们将看到用于 [Internet Socket](https://en.wikipedia.org/wiki/Berkeley_sockets) 的 Socket API，有时称为 Berkeley 或 BSD Socket。当然也有 [Unix domain sockets](https://en.wikipedia.org/wiki/Unix_domain_socket) —— 一种用于 **同一主机** 进程间的通信

