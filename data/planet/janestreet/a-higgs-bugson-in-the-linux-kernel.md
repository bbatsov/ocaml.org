---
title: A Higgs-bugson in the Linux Kernel
description: "We recently ran across a strange higgs-bugson that manifested itself
  in a critical system that stores and distributes the firm\u2019s trading activity
  data, calle..."
url: https://blog.janestreet.com/a-higgs-bugson-in-the-linux-kernel/
date: 2025-07-02T00:00:00-00:00
preview_image: https://blog.janestreet.com/a-higgs-bugson-in-the-linux-kernel/prod_vs_test.apng
authors:
- Jane Street Tech Blog
source:
---

<p>We recently ran across a strange higgs-bugson that manifested itself in a critical system that stores and distributes the firm’s trading activity data, called Gord. (A <a href="https://en.wikipedia.org/wiki/Heisenbug#Related_terms">higgs-bugson</a> is a bug that is reported in practice but difficult to reproduce, named for the <a href="https://en.wikipedia.org/wiki/Search_for_the_Higgs_boson">Higgs boson</a>, a particle which was theorized in the 1960s but only found in 2013.) In this post I’ll walk you through the process I took to debug it. I tried to write down relevant details as they came up, so see if you can guess what the bug is while reading along.</p>


