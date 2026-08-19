---
title: "Linux kernel Spring Unpaid 2026"
date: 2026-08-14T19:17:33-03:00
categories: ["linux kernel"]
description: "Mentee at Linux kernel Spring Unpaid 2026"
---

#### Linux kernel Spring Unpaid 2026
Very happy to share that in 2026 I got selected for a 6-month unpaid Linux Kernel [mentorship](https://mentorship.lfx.linuxfoundation.org/project/53378ec5-48d7-4c49-a01f-8cbd3948db3d) program provided by the Linux Foundation:

<img style="display: block; margin: auto;" src="/mentorship.png"/><br/>

_"We have received 693 applications, 159 people submitted completed
applications, and you have made it through!"_

...
The reason I applied for it (and I'm very grateful for being selected) was that I still have a particular taste for open source and a feeling that I could do more with what I've learned throughout my professional journey so far.

It's worth mentioning that anyone can apply for this mentorship — the requirements bar is easier to meet: you just need to, at the very least, complete a very basic Linux Foundation course about the Linux Kernel. Also, you don't need to have any previous patches submitted, but if you have some, it could be some extra juice towards being selected for the program.

#### What is the program

The program has formal mentors, and in this case, it was none other than Shuah Khan, a well-known and experienced person in the community (and across the Linux Foundation), along with Brigham Campbell, another great professional with a lot of Linux Kernel expertise.

The purpose of the program was to introduce more people to the community and help them start their journey — if they want to — contributing to the project.

#### How it was for me

The mentorship was divided into two "stages" I would say. The first one was regular weekly meetings with the other mentees and mentors for broader discussions about the Linux Kernel, like how the submission process works, merge windows, and how to follow up on patches on lore, as well as best practices for interacting with the community and maintainers, which can sometimes be difficult for beginners.

For me, at the time of the mentorship, I already had some experience with the kernel (in a few subsystems) and a few patches already applied upstream. Even so, it was very productive to be part of the meetings and internal discussions, because people were raising points I had struggled to find answers to during my initial steps in the community, so those different visions and perspectives added a lot of value and experience.

The second "stage" was that mentees should select a subsystem of interest, as much as possible, so they could start searching for possible areas for initial contributions. Of course, you won't necessarily find things to do in a single subsystem, so mentees had the flexibility to look at different areas as well. 

It's worth saying (and these are my own words) that even after
selecting an area, it's hard to find something to contribute that's feasible, because you have "two sides of a coin" here:
- you have some hardware (Raspberry Pi, Banana Pi, etc.) with a set of sensors whose drivers you could explore;
- you don't have that, so you need to find something else you could work on, and maybe use emulation like QEMU (not mandatory, depending on what you are doing/changing);

The first case is easier: you can change something, compile it, and test it directly to see how it behaves. The latter is trickier, since you may or may not need to emulate a different arch; you might need an extra rootfs with an init process (or, let's say, with busybox binaries) to be able to boot and interact with the kernel. 

For both cases, there are a few methods for finding something to work on: [syzkaller](https://syzkaller.appspot.com/upstream), searching for `TODO` files, `grep`-ing for specific keywords or breadcrumbs like `TODO`/`FIXME`/`XXX` in the source files, or even bad practices like returning improper `errno` numbers that aren't semantically correct (like my first patch that replaced hardcoded return [values](https://web.git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=00ea2b0dc6ff47e3d3d976fd788aa22373d042b8)), `probe` functions not cleaning up resources, using checkpatch, or even "static analysis" tools like [smatch](https://smatch.sourceforge.net/) for code inspection or even hardware descriptions in the device tree binding area, etc.

#### Contributions during mentorship

Apart from having a Raspberry Pi 4 in hand, I used the strategy of scanning and looking for something to do without actually needing the hardware itself. The result of that was a few submissions in staging (my first subsystem "choice"), iio and device tree area:

```bash
// Against linux-next:
$ pwd
/home/gobbi/workspace/linux-next
$ date
Tue Aug 18 07:50:26 PM -03 2026

$ git log --pretty --format=oneline --author=gobbi
ac4c8003f152d52b21af06aeb28a1ee1d9348568 staging: rtl8723bs: fix unnamed parameters warning detected at checkpatch
f6ae81a98a7b72674d4ffef8daa87f4c958dd92b iio: adc: spear_adc: align headers with IWYU principle
2fab7499006ddc87c8937b149b769b8eb4a65217 iio: adc: spear_adc: sort includes alphabetically
6671dbbb12513e79ccaccbf799b7b56ae28bb20f staging: rtl8723bs: remove unused arg at odm_interface.h
...(patches from before the program)
```
More details about the ones already applied:
- [staging: rtl8723bs: remove unused arg at odm_interface.h](https://web.git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=6671dbbb12513e79ccaccbf799b7b56ae28bb20f)
- [iio: adc: spear_adc: sort includes alphabetically](
https://web.git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2fab7499006ddc87c8937b149b769b8eb4a65217)
- [iio: adc: spear_adc: align headers with IWYU principle](https://web.git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=f6ae81a98a7b72674d4ffef8daa87f4c958dd92b)
- [staging: rtl8723bs: fix unnamed parameters warning detected at checkpatch](
https://git.kernel.org/pub/scm/linux/kernel/git/gregkh/staging.git/commit/?h=staging-next&id=ac4c8003f152d52b21af06aeb28a1ee1d9348568)

A few that are "ongoing":
- [PATCH v4 0/3 staging: media: atomisp: use kvmalloc_objs() and drop redundant OOM messages](https://lore.kernel.org/all/20260714225235.47134-1-rodrigo.gobbi.7@gmail.com/)
- [PATCH v5 0/2 iio: proximity: move LIDAR-Lite out of trivial-devices and add Garmin fallback](https://lore.kernel.org/all/20260810192131.153495-1-rodrigo.gobbi.7@gmail.com/#t)

And another one that was dropped during a broader discussion about `dev_warn()` usage:
- [PATCH iio: imu: bmi270: use dev_warn for unexpected chip id](https://lore.kernel.org/all/20260316232007.22887-1-rodrigo.gobbi.7@gmail.com/)

#### Final thoughts
Someone asked me some time ago, "What defines something as being accepted into mainline?" Well, this program reinforced a few important aspects present in our lives as software engineers, such as:
- Why are we changing something?
- What are you trying to fix or improve?
- Can you prove that you're not going to break something, or at least be transparent that you won't be able to test it with real hardware?
- You can be very technical with a lot of expertise, but can you explain and defend your point of view clearly and directly about what you're changing?
- Can you keep things simple and robust?

And a lot of other reasoning resulted from this program and from the discussions during the meetings. I will end up saying something that I always discuss with colleagues, and I appreciate people who have that "skill" (and I include myself in this): you need to be open to learning something new and improving yourself in any way. This mentorship showed me, again, that this is always the case: the exchange with mentors and mentees, and with the maintainers during patch review rounds, was all very productive and I hope to keep applying everything to my journey.
