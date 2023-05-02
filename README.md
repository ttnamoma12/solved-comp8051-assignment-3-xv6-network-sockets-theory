Download Link: https://assignmentchef.com/product/solved-comp8051-assignment-3-xv6-network-sockets-theory
<br>
<h3>Question 1 – Parse network packets from the E1000 driver</h3>

Write a parse function in the file e1000.c in the code in network-sockets-xv6e1000-lab.zip to give a human readable dump of received packet details.

Your parse function should distinguish between UDP, ARP, TCP, packets. For the MAC layer your parse function should detail the mac addresses and the ethertype field. For the IP layer your parse function should detail the source and destination ip addresses and the flags.

<strong><em> (9 marks)</em></strong>

<strong>Background</strong>

Download the network-sockets-xv6-e1000-lab.zip from Canvas.

The code is taken from here  <a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html">https://pdos.csail.mit.edu/6.828/2019/labs/net.html </a>Note the code at this link <a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html">  </a><a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html">https://pdos.csail.mit.edu/6.828/2019/labs/net.htm</a> <a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html">l</a> <u> </u>is for the 2019 version of xv6 i.e. riscv. See canvas for a port of the code to xv6 for the x86 i.e. network-sockets-xv6-e1000-lab.zip.

You may find it helpful to review “Traps and device drivers”, ” File descriptor layer” from the xv6 book, and <a href="https://pdos.csail.mit.edu/6.828/2019/lec/l-networking.pdf">the lecture notes on networking</a><a href="https://pdos.csail.mit.edu/6.828/2019/lec/l-networking.pdf">.</a>

We are using a virtual network device called the E1000 to handle network communication. To xv6, the E1000 looks like a real piece of hardware connected to a real Ethernet local area network (LAN). But in reality, the E1000 that the  driver talks to is an emulation provided by qemu, connected to a LAN that is also emulated by qemu. On this LAN, xv6 (the “guest”) has an IP address of 10.0.2.15. The only other (emulated) computer on the LAN has IP address

10.0.2.2. qemu arranges that when xv6 uses the E1000 to send a packet to 10.0.2.2, it’s really delivered to the appropriate application on the (real) computer on which you’re running qemu (the “host”).

We will be using QEMU’s user mode network stack since it requires no administrative privileges to run. QEMU’s documentation has more about user-net <a href="https://wiki.qemu.org/download/qemu-doc.html#Using-the-user-mode-network-stack">here</a><a href="https://wiki.qemu.org/download/qemu-doc.html#Using-the-user-mode-network-stack">.</a> We’ve updated the Makefile to enable QEMU’s user-mode network stack and the virtual E1000 network card.

QEMU’s network stack will record all incoming and outgoing packets to packets.pcap. To get a hex/ASCII dump of captured packets use tcpdump like this:

tcpdump -XXnr packets.pcap or use wireshark wireshark packets.pcap

<strong>Instructions</strong>

<ol>

 <li>Download the code from canvas – network-sockets-xv6-e1000-lab.zip</li>

 <li>See the slides qemu-ethernet on canvas and associated video for details aboutvarious the network layer packet formats.</li>

 <li>You are to write the parse function in e1000.c.</li>

 <li>To test your parse code for TCP, ARP – use the browser to connect to <a href="http://localhost:20001/">http://localhost:20001/</a></li>

 <li>To test your parse code for UDP. See net.h for a c struct to represent an UDP packet. See also the details in question 2.</li>

</ol>

<h3>Question 2 –  Socket layer xv6</h3>

Read the description below and also the “<em>File descriptor layer</em>” section in the xv6 book and describe how the user is able to send and receive packets to/from the E1000 device with simple system calls such as <strong>read </strong>and <strong>write</strong>.

<strong><em> (9 marks) </em>Overview of the socket layer in xv6</strong>

Download the network-sockets-xv6-e1000-lab.zip from Canvas. The code is taken from here  <a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html"> </a><a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html">https://pdos.csail.mit.edu/6.828/2019/labs/net.htm</a> <a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html">l</a> <a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html">.</a> Note the code at this link  <a href="https://pdos.csail.mit.edu/6.828/2019/labs/net.html">https://pdos.csail.mit.edu/6.828/2019/labs/net.html</a>  is for the 2019 version of xv6 i.e. riscv. See canvas for a port of the code to xv6 for the x86 i.e. network-sockets-xv6-e1000-lab.zip.

Network sockets are a standard abstraction for OS networking that bear similarity to files. Sockets are accessed through ordinary file descriptors (just like files, pipes, and devices). Reading from a socket file descriptor receives a packet while writing to it sends a packet. If no packets are currently available to be received, the reader must block and wait for the next packet to arrive (i.e. allow rescheduling to another process). The code in xv6-e1000-sockets.zip  is a stripped down version of sockets that supports the UDP network protocol.

Each network socket only receives packets for a particular combination of local and remote IP addresses and port numbers, and xv6 is required to support multiple sockets. A socket can be created and bound to the requested addresses and ports via the connect system call, which returns a file descriptor. The implementation of this system call is in kernel/sysfile.c. The code for sockalloc() and related functions is in kernel/sysnet.c.

Take note of the provided data structures; one struct sock object is created for each socket. sockets is a singly linked list of all active sockets. It is useful for finding which socket to deliver newly received packets to. In addition, each socket object maintains a queue of mbufs waiting to be received. Received packets will stay in these queues until the read() system call dequeues them.

<strong>Running the test program.</strong>

(in one terminal on your laptop) $ python2 server.py 26099 listening on localhost port 26099

(then on xv6 in another terminal on the same machine run make qemu

and run nettests at the xv6 shell prompt

$ nettests

<h3>Question 3 –  Firecracker, virtio and virtio-sock</h3>

Give an overview of firecracker and its use of virtio for networking. In your answer focus specifically on virtio-sock.

<strong><em> (12 marks) </em></strong>References: <a href="https://www.usenix.org/system/files/nsdi20-paper-agache.pdf">https://www.usenix.org/system/files/nsdi20-paper-agache.pdf</a>  <a href="https://github.com/firecracker-microvm/firecracker">https://github.com/firecracker-microvm/firecracker </a><a href="https://github.com/firecracker-microvm/firecracker/blob/master/docs/vsock.md">https://github.com/firecracker-microvm/firecracker/blob/master/docs/vsock.md </a><a href="https://www.youtube.com/watch?v=LFqz-VZPhFE">https://www.youtube.com/watch?v=LFqz-VZPhFE </a><a href="https://terenceli.github.io/%E6%8A%80%E6%9C%AF/2020/04/18/vsock-internals">https://terenceli.github.io/%E6%8A%80%E6%9C%AF/2020/04/18/vsock-internals </a><a href="https://www.youtube.com/watch?v=R5DQWdPUQSY">https://www.youtube.com/watch?v=R5DQWdPUQSY</a>

<a href="https://developer.ibm.com/technologies/linux/articles/l-virtio/">https://developer.ibm.com/technologies/linux/articles/l-virtio/</a>