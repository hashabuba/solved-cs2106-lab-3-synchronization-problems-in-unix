Download Link: https://assignmentchef.com/product/solved-cs2106-lab-3-synchronization-problems-in-unix
<br>
In this lab, you’ll be solving two synchronisation problems (split into Sections 2 and 3 below) by implementing your own constructs.  In Section 2, you are only allowed to use semaphores; but in Section 3, you are allowed to use any synchronisation mechanism (except busy waiting).




To help you focus on the implementation of synchronisation mechanisms, a driver file (named ex&lt;#&gt;.c) has been provided for you.  The driver file contains the main() function, and handles reading from the input file, spawning all the threads, and printing out the events that occur.  Your task will be implementing functions that the driver calls.

<strong> </strong>

<strong>           </strong>

<strong>Lab directory structure:</strong>




The skeleton code for each exercise is placed in a separate folder.  In Section 2, the driver code for exercises 1 and 2 are identical.  In Section 3, the driver code for exercises 4, 5, and 6 are identical.




<h2><strong>Test cases in this writeup </strong></h2>




In this writeup, you will find many test cases for the various exercises.  Input text are left-aligned, output text are right-aligned, and parenthesised text in small font are comments (that are not visible when running your program).




<h2><strong>Reminder for Mac users </strong></h2>




Please note that POSIX semaphores do not work on macOS.  Compiling programs with semaphores generally does not throw an error, but the semaphore functions will silently fail.  We strongly recommend that you work on this lab using the SoC Compute Cluster.

<strong> </strong>




<h1>Section 2. Ball Packing</h1>

<strong> </strong>

There are a total of <strong>three exercises</strong> (ex1 to ex3) in this section.  <strong>You may only use POSIX semaphores (i.e., those in &lt;semaphore.h&gt;, such as sem_wait()/sem_post()) for this section.  You are not allowed to use any other synchronisation mechanism, including those in &lt;pthread.h&gt;, nor busy waiting. </strong>




<h2>2.1. Problem Setup</h2>




You are the manager of a factory that manufactures coloured balls.  Each ball comes in one of three colours – red, green, or blue.  The balls are to be packed into boxes of N balls each (where N is at least 2), and all balls in a box must have the same colour.  The balls enter the packing area at arbitrary times, and they should stay at the packing area until there are N balls of the same colour.  When there are N balls of the same colour, those N balls should be immediately released (at the same time) from the packing area.




Each ball has a <strong>unique </strong>id and is modelled as a thread. You are to implement a synchronisation mechanism to block each ball until it can be released from the packing area.




In each exercise, you will find the following files:

<table width="553">

 <tbody>

  <tr>

   <td width="163"><strong>Makefile</strong><strong>(do not modify)</strong></td>

   <td width="390">Used to compile your files. Just run make.</td>

  </tr>

  <tr>

   <td width="163"><strong>ex&lt;#&gt;.c</strong> <strong>(do not modify)</strong></td>

   <td width="390">Prewritten driver file.(It will be replaced when grading)</td>

  </tr>

  <tr>

   <td width="163"><strong>packer.h</strong>  <strong>(do not modify)</strong></td>

   <td width="390">Prewritten header file. Defines function prototypes.</td>

  </tr>

  <tr>

   <td width="163"><strong>packer.c</strong></td>

   <td width="390">Implement your synchronisation mechanisms here.</td>

  </tr>

 </tbody>

</table>




You should only modify <strong>packer.c</strong>.  Changes to all other files will be discarded, and the driver file will be replaced with a different version for grading.




<h3>Compiling and running the exercises</h3>

<strong> </strong>

To compile the exercises, run <strong>make </strong>in the <em>ex1-3 </em>subdirectory. You will not be penalized for warnings, but you are strongly encouraged to resolve all warnings. You are also advised to use <em>valgrind </em>to make sure your program is free of memory errors.




To run the exercises, simply run: <strong>$ ./ex[1, 2, or 3] &lt; [test program] </strong>where [test program] is an input file for the driver program. Please see the next section for the expected input format. You can also use <em>stdin </em>to input commands manually.




<strong>           </strong>

<h3>Driver input format</h3>




The driver program takes input from stdin. You are encouraged to write test commands into a file, and use input redirection to feed the test file to the driver program. We have also provided several sample test files for each exercise. The driver program expects the following input format:




The first line contains a single integer, N, specifying the number of balls in each box (N &gt;= 2).




Subsequent lines are in one of the following forms:

<ul>

 <li>&lt;<em>colour</em>&gt; &lt;<em>id</em>&gt; – this is a command indicating that a ball with the given <em>colour </em>and <em>id </em>has arrived at the packing area</li>

 <li>. – a literal period indicates a synchronisation point for the driver (see below for details)</li>

</ul>




You can make the following <strong>assumptions </strong>about the input we will use for grading:

<ul>

 <li>It is guaranteed that at the end of the input, all balls can be packed (i.e., there should be no balls remaining in the packing area).</li>

 <li>All balls in one simulation run will have <strong>unique </strong> – The input file is less than 1000 lines long.</li>

</ul>




<h3>Parallelism in the driver</h3>




As per the input format above, input commands (i.e., balls arriving in the packing area) are separated by synchronisation points. The driver batches together all commands until it encounters a synchronisation point or EOF, and then it executes those commands simultaneously in parallel (each ball gets its own thread).  The driver is designed to execute those commands at maximum concurrency.




<h3>Nondeterminism</h3>




Note that the output of your program may be nondeterministic when multiple balls arrive at the same time.  This is because when multiple balls arrive at the same time, any one of them might be processed first (e.g., to complete a box).







<h2>2.2. Exercise 1</h2>




In this first exercise, we impose a few more constraints on the problem setup to simplify your implementation:

<ul>

 <li>N = 2 (i.e., balls are packed in pairs)</li>

 <li>There will be <em>at most two balls of each colour</em> in the entire simulation.</li>

</ul>




Note that this means that there will be at most 6 balls in the entire simulation, and at most one box of balls of each colour.




<u>Functions you need to implement:</u>

<ul>

 <li>void packer_init(void);</li>

</ul>

This function is called once at the start of the program.  Use this function to perform any necessary setup (e.g., allocate global variables and semaphores).

<ul>

 <li>void packer_destroy(void);</li>

</ul>

This function is called once at the end of the program.  Use this function to free allocated resources that persist throughout the simulation.

<ul>

 <li>int pack_ball(int colour, int id);</li>

</ul>

This function is called when a ball enters the packing area, giving the colour and id of this ball.  The colour is an integer between 1 and 3 inclusive, giving the colour of the ball (encoded as an integer).  The id of a ball may be any integer.  This function should block until there is another ball of the same colour entering the area. When that happens, this function should return the id of the <strong>other ball</strong> that should be packed with it in the same box.




As stated in the assumptions, it is guaranteed that no two balls will have the same id.  However, ids are arbitrary and may not be issued monotonically.




Below are some test cases to help you understand the requirements and test your code. However, you are strongly encouraged to come up with more test cases. Note that due to non-determinism, your output may differ from the example runs below.




<strong>           </strong>

<h3>Example sequential test case (ex1/seq_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="229">Input</td>

   <td width="324">Possible output</td>

  </tr>

  <tr>

   <td width="229">21      14 .2      12 .2 12345 .</td>

   <td width="324">Ball 12 was matched with ball 12345Ball 12345 was matched with ball 12</td>

  </tr>

  <tr>

   <td width="229">3 333 .1 25 .</td>

   <td width="324">Ball 25 was matched with ball 14Ball 14 was matched with ball 25</td>

  </tr>

  <tr>

   <td width="229">3 87878(Ctrl+D pressed to end input stream)</td>

   <td width="324">Ball 333 was matched with ball 87878 Ball 87878 was matched with ball 333</td>

  </tr>

 </tbody>

</table>




Since there is a period (‘.’) separating every command, all the commands are executed sequentially, allowing balls to be matched (if any) before issuing the next command.




The line “Ball X was matched with ball Y” indicates that the <em>pack_ball()</em> invocation by ball X has returned and your synchronisation mechanism has decided that balls X and Y should go into the same box.




Note: The output for each of the two balls in a matched pair may be printed in either order. Both outputs are correct.




Note 2: The driver will pause for 100ms after each ‘.’ to wait for balls to be matched.  There is a possibility that 100ms is not long enough for the balls to be matched, especially if your system has a high load. Based on our testing on the SoC compute cluster, we do not expect this to happen.  In the unlikely event that this actually happens, you can try increasing the time limit of the usleep() call in the driver.  You should also check your solutions for deadlocks, as they could be a reason for not seeing the desired output.




Note 3: The very first integer in the input is the value of N, and it is always “2” for this exercise.




<strong>           </strong>

<h3>Example parallel test case (ex1/par_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="247">Input</td>

   <td width="306">Possible output</td>

  </tr>

  <tr>

   <td width="247">21 1801      3352      121 .</td>

   <td width="306">Ball 335 was matched with ball 180Ball 180 was matched with ball 335</td>

  </tr>

  <tr>

   <td width="247">3 456 .2  4553  457(Ctrl+D pressed to end input stream)</td>

   <td width="306">Ball 121 was matched with ball 455Ball 456 was matched with ball 457Ball 457 was matched with ball 456Ball 455 was matched with ball 121</td>

  </tr>

 </tbody>

</table>




Commands that are not separated by periods are batched together, meaning that pack_ball() will be called on each arriving ball concurrently.  You need to ensure that your synchronisation mechanism handles these cases.




<h3>Example tiny test case (ex1/tiny_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="282">Input</td>

   <td width="271">Possible output</td>

  </tr>

  <tr>

   <td width="282">21 1 .1 2(Ctrl+D pressed to end input stream)</td>

   <td width="271">Ball 1 was matched with ball 2 Ball 2 was matched with ball 1</td>

  </tr>

 </tbody>

</table>




This test case highlights that it is not necessary to receive exactly two balls of each colour.  It could be possible to receive zero balls of some colours. Note that it is impossible to only receive one ball of some colour, because that would violate the guarantee that all balls can be packed at the end of the simulation.







<h2>2.3. Exercise 2</h2>




In this exercise, we still have the constraint that N = 2.  However, there may now be more than two balls of each colour. As such, you will need to be careful about which balls, amongst those of the same colour, get packed. In particular, a ball that arrives earlier than another ball of the same colour must be packed no later than that other ball. (Remember that when we say that ball A “arrives earlier” than ball B, it means that the command for ball A appears before the command for ball B and is separated by at least one synchronisation point.)




As this exercise is a superset of exercise 1, all test cases from exercise 1 are also valid for exercise 2.




The driver and header files for exercise 2 are identical to those from exercise 1.




Here is an example to illustrate this requirement:




<h3>Example ordering test case (ex2/order_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Input                                                                                    Possible output</td>

  </tr>

  <tr>

   <td width="553">21 101 .1 1021 103 .(Since ball 101 arrived before balls 102 and 103, ball 101 must be packed with one of them. Either ball 102 or ball 103 may be chosen to be packed with ball 101.)Ball 101 was matched with ball 102Ball 102 was matched with ball 1011 1041 105 .(The remaining unpaired ball (ball 103) must be packed with either ball 104 or 105.)Ball 103 was matched with ball 105Ball 105 was matched with ball 1031 1061 1071 108(Ctrl+D pressed to end input stream)(The remaining unpaired ball (ball 104) may be paired with any of the three new balls.)Ball 106 was matched with ball 108Ball 108 was matched with ball 106Ball 104 was matched with ball 107Ball 107 was matched with ball 104</td>

  </tr>

 </tbody>

</table>




Below are some other test cases to check your work.  You are strongly encouraged to test your code with more test cases on your own.

<strong> </strong>

<strong>           </strong>

<h3>Example sequential test case (ex2/seq_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="229">Input</td>

   <td width="324">Possible output</td>

  </tr>

  <tr>

   <td width="229">21      123 .2      11 .1 456 .</td>

   <td width="324">Ball 123 was matched with ball 456Ball 456 was matched with ball 123</td>

  </tr>

  <tr>

   <td width="229">2 1000 .</td>

   <td width="324">Ball 11 was matched with ball 1000Ball 1000 was matched with ball 11</td>

  </tr>

  <tr>

   <td width="229">3 145 .3 144 .</td>

   <td width="324">Ball 145 was matched with ball 144Ball 144 was matched with ball 145</td>

  </tr>

  <tr>

   <td width="229">3 2000 .2 2001 .1 2002 .3 2003 .</td>

   <td width="324">Ball 2000 was matched with ball 2003Ball 2003 was matched with ball 2000</td>

  </tr>

  <tr>

   <td width="229">2 43 .</td>

   <td width="324">Ball 2001 was matched with ball 43Ball 43 was matched with ball 2001</td>

  </tr>

  <tr>

   <td width="229">1 5(Ctrl+D pressed to end input stream)</td>

   <td width="324">Ball 2002 was matched with ball 5 Ball 5 was matched with ball 2002</td>

  </tr>

 </tbody>

</table>




<strong>           </strong>

<h3>Example parallel test case (ex2/par_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="247">Input</td>

   <td width="306">Possible output</td>

  </tr>

  <tr>

   <td width="247">21      1012      1023      103 .1 1043 105 .</td>

   <td width="306">Ball 103 was matched with ball 105Ball 101 was matched with ball 104Ball 104 was matched with ball 101Ball 105 was matched with ball 103</td>

  </tr>

  <tr>

   <td width="247">2 1062 107 .</td>

   <td width="306">Ball 102 was matched with ball 107Ball 107 was matched with ball 102</td>

  </tr>

  <tr>

   <td width="247">2 1082 109 .</td>

   <td width="306">Ball 106 was matched with ball 109Ball 109 was matched with ball 106</td>

  </tr>

  <tr>

   <td width="247">2 1102 1111 1123 1131 114 .</td>

   <td width="306">Ball 108 was matched with ball 110Ball 110 was matched with ball 108Ball 114 was matched with ball 112Ball 112 was matched with ball 114</td>

  </tr>

  <tr>

   <td width="247">3 115 .</td>

   <td width="306">Ball 113 was matched with ball 115Ball 115 was matched with ball 113</td>

  </tr>

  <tr>

   <td width="247">2 116(Ctrl+D pressed to end input stream)</td>

   <td width="306">Ball 111 was matched with ball 116 Ball 116 was matched with ball 111</td>

  </tr>

 </tbody>

</table>







<h2>2.4. Exercise 3</h2>




In this exercise, we no longer have the constraint that N = 2.  This means that the number of balls to be packed in each box may be any integer <em>at least</em> 2.  For grading purposes, we guarantee that N is no more than 64. POSIX semaphores on the SoC Compute Cluster can be incremented to a value of more than 64 without issue, so you do not need to worry about semaphore limits.




As this exercise is a superset of exercise 2, all test cases from exercises 1 and 2 are also valid for exercise 3. You are recommended to test your code with those test cases as well.




There are some changes to the API between the driver and your code, to allow returning the ids of <em>all</em> the other balls to be packed together. Specifically, the pack_ball() function now has the following signature:

<ul>

 <li>void pack_ball(int colour, int id, int *other_ids);</li>

</ul>

The <em>colour </em>and <em>id</em> of the arriving ball are provided to you in the same way. The other_ids parameter points to an array of length N−1, and you should write the ids of the N−1 other balls that should be packed with this ball into the array before returning from this function.  Those

N−1 ids may be written in any order.  (Note that the array has length N−1 because you do not include the id of the current ball itself.)




Notice that instead of returning the id of the other ball via the return value, we are now returning the ids of the other balls via an output array parameter. Note that like previous exercises, you can still assume that all balls have <strong>unique </strong>ids. Also note that <em>other_ids</em> points to memory that is owned by the driver – you do not need to allocate extra memory to pass the N−1 ids.




The driver for exercise 3 is modified from that of exercise 2, to handle the varying value of N and the different API.  However, the behaviour of this driver is very similar to that of the previous exercises.




<h3>Example test case (ex3/test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="150">Input</td>

   <td width="403">Possible output</td>

  </tr>

  <tr>

   <td width="150">41 1231 234 .1 1113 5611 3231      888 .2      606</td>

   <td width="403">Ball 123 was matched with balls 234, 323, 888Ball 234 was matched with balls 323, 888, 123Ball 323 was matched with balls 234, 888, 123Ball 888 was matched with balls 234, 323, 123</td>

  </tr>

  <tr>

   <td colspan="2" width="553">2 6072 6082 6092 6102 6112 6122 6132 6142       615 .Ball 611 was matched with balls 614, 610, 615Ball 610 was matched with balls 614, 615, 611Ball 615 was matched with balls 614, 610, 611Ball 614 was matched with balls 610, 615, 611Ball 607 was matched with balls 609, 613, 612Ball 613 was matched with balls 609, 612, 607Ball 612 was matched with balls 609, 613, 607Ball 609 was matched with balls 613, 612, 607 3 6163       432 .1       700 .2       7081 705 .1  3602  3703  380(Ctrl+D pressed to end input stream)Ball 561 was matched with balls 432, 616, 380Ball 608 was matched with balls 606, 708, 370Ball 616 was matched with balls 432, 380, 561Ball 606 was matched with balls 708, 370, 608Ball 111 was matched with balls 700, 705, 360Ball 708 was matched with balls 606, 370, 608Ball 432 was matched with balls 616, 380, 561Ball 705 was matched with balls 700, 360, 111Ball 370 was matched with balls 606, 708, 608Ball 700 was matched with balls 705, 360, 111Ball 380 was matched with balls 432, 616, 561Ball 360 was matched with balls 700, 705, 111</td>

  </tr>

 </tbody>

</table>




Note that the number of balls per box, N, is given at the very start of the input file, and it is fixed for the entire simulation.




<h1>Section 3. Restaurant</h1>

<strong> </strong>

There are a total of <strong>three exercises</strong> (ex4 to ex6) in this section.  Exercise 6 is a bonus exercise.  In this section, you may use any synchronisation primitives from &lt;semaphore.h&gt; and &lt;pthread.h&gt;. This includes pthread mutexes, barriers, and condition variables. However, it is possible to solve all exercises with only semaphores.




Note that you are not allowed to use busy waiting.




<h2>3.1. Problem Setup</h2>




You operate a busy restaurant in the populous city of Singapore. There are many tables in your restaurant, and each table has 1, 2, 3, 4, or 5 seats.  Groups of people (where each group has 1, 2, 3, 4, or 5 people) queue up outside your restaurant, waiting to be seated. You need to assign a table to each arriving group or keep them in the queue if there are no available tables for them. After a group has finished their meal, they will vacate the table that they have been assigned, and you can assign the vacated table to another group in the queue.




Each of the three exercises in this section comes with varying rules for how tables are to be assigned to groups.  Each group is modelled as a thread, and you are to implement a synchronisation mechanism to queue the groups up and assign them to tables.




In each exercise, you will find the following files:

<table width="553">

 <tbody>

  <tr>

   <td width="163"><strong>Makefile</strong><strong>(do not modify)</strong></td>

   <td width="390">Used to compile your files. Just run make.</td>

  </tr>

  <tr>

   <td width="163"><strong>ex&lt;#&gt;.c</strong> <strong>(do not modify)</strong></td>

   <td width="390">Prewritten driver file.(It will be replaced when grading)</td>

  </tr>

  <tr>

   <td width="163"><strong>restaurant.h</strong></td>

   <td width="390">Prewritten header file. You may add fields to the struct <em>group_state</em> definition, but you are not to modify the function declarations.</td>

  </tr>

  <tr>

   <td width="163"><strong>restaurant.c</strong></td>

   <td width="390">Implement your synchronisation mechanism here.</td>

  </tr>

 </tbody>

</table>




You synchronisation mechanism should be implemented in <strong>restaurant.c</strong>.  You may add fields to the struct <em>group_state</em> in <strong>restaurant.h</strong>.  Changes to all other files will be discarded, and the driver file will be replaced with a different version for grading.




All three exercises in this section have identical drivers and skeleton code.




<h3>Functions you need to implement</h3>




The functions that you need to implement are identical for all exercises of this section.  However, the required behaviours of the functions are different, and will be explained in each exercise.

<strong> </strong>




<u>You need to implement the following four functions:</u>

<ul>

 <li>void restaurant_init(int num_tables[5]);</li>

</ul>

This function is called once at the start of the program, providing you with the number of tables with each number of seats.  For each <em>i</em> in {1, 2, 3, 4, 5}, <em>num_tables</em>[<em>i</em>-1] is the number of tables with <em>i</em> seats.  Use this function to perform any setup necessary (e.g., allocate global arrays). You can assume all numbers in the array are positive integers.

<ul>

 <li>void restaurant_destroy(void);</li>

</ul>

This function is called once at the end of the program.  Use this function to free allocated resources that persist throughout the entire simulation.

<ul>

 <li>int request_for_table(group_state *state, int num_people);</li>

</ul>

This function is called when a group enters the queue of your restaurant, giving the number of people in that group, which is an integer between 1 and 5 inclusive. This function should block until there is an available table that can be assigned to this group (according to exercise-specific rules that will be explained later).  When that happens, this function should return the id of the table assigned to this group. Furthermore, once this group is in the queue, but before blocking, you need to call the on_enqueue() function (implemented by the driver) – see below for details.  You can record information in <em>state </em>(you are allowed to add fields to <em>group_state</em> in <em>restaurant.h</em>); the same <em>state</em> pointer will be passed to the leave_table() invocation when the group leaves the table.

<ul>

 <li>void leave_table(group_state *state);</li>

</ul>

This function is called when a group that is currently seated at a table leaves the restaurant.  At this point, you should process the queue to see if any group can by assigned to the table vacated by this group.  This function should not block.




The request_for_table() and leave_table() functions are called when you issue specific input commands to the driver.  The driver input format will be clarified later in more detail.

<strong> </strong>

<h3>Queueing</h3>




Groups queue up in a line to enter the restaurant.  <strong>A group may only be assigned a table if no other group in front of this group in the queue can be assigned a table.</strong>  This means that a group may jump the queue only if all groups in front of it are unable to be assigned a table.  The examples in each exercise will make this requirement clear. Keep in mind that semaphore (and other POSIX synchronization primitive) implementations do not guarantee the order in which blocking threads are woken up, so you need to implement your own mechanisms to enforce this ordering.




<strong>           </strong>

<h3>Queue notification</h3>




For the purpose of grading, our grader needs to know the order in which the groups are queued.  However, if our grader calls request_for_table() on two groups one after another and those two calls block, our grader will be unable to tell if the first group indeed queued up before the second group.  This is because our grader does not know how long it should wait before sending the second group – the grader cannot be sure how long your synchronisation mechanism takes to block.




To resolve this situation, we have implemented the on_enqueue() function that takes in no parameters and returns void.  In your implementation of request_for_table(), you are <strong>required </strong>to call on_enqueue() (exactly once) <strong>before blocking</strong>, once this group gets a position in the queue.  More formally, given two groups, A and B, if group B invokes request_for_table() <em>after</em> group A’s invocation of on_enqueue(), then group B must be behind group A in the queue.  Our grader will wait until all arriving groups call the on_enqueue() function before issuing the next batch of commands (so if you do not call on_enqueue(), our grader will not issue any more commands and there will be a deadlock).




You <strong>do not</strong> need to synchronise calls to on_enqueue(), as the driver implements its own synchronisation mechanism for printing log messages.  You should call on_enqueue() even if the group does not need to wait (i.e., there is an available table that can be immediately assigned to this group).




<h3>Compiling and running the exercises</h3>

<strong> </strong>

To compile the exercises, run <strong>make </strong>in the <em>ex4-6 </em>subdirectories.  You will not be penalized for warnings, but you are strongly encouraged to resolve all warnings.




To run the exercises, simply run:

<h3>$ ./ex[4, 5, or 6]</h3>

and type in (or paste) batches of commands.  Please refer to the next section for the expected input format.  Due to nondeterminism (see the <em>Interactivity</em> section later), the validity of a command may depend on the response of previous commands.




Note that Valgrind might take a long time to run, because it is essentially a virtual machine that emulates every thread synchronously.




<h3>Driver input format</h3>

<strong> </strong>

Similar to the ball packing exercises, the driver program takes input from stdin. The driver program expects the following input format:




The first line contains five <strong>positive</strong> (i.e., strictly greater than zero) integers n<sub>1</sub>, n<sub>2</sub>, n<sub>3</sub>, n<sub>4</sub>, and n<sub>5</sub>, representing the number of tables with 1, 2, 3, 4, and 5 seats respectively.  The five integers are separated by space.  Tables are (implicitly) given ids starting from 0 – tables with 1 seat are given ids 0 to n<sub>1</sub>−1 inclusive, tables with 2 seats are given ids n<sub>1</sub> to n<sub>1</sub>+n<sub>2</sub>−1 inclusive, and so on.  Hence, the table ids overall range from 0 to n<sub>1</sub>+n<sub>2</sub>+n<sub>3</sub>+n<sub>4</sub>+n<sub>5</sub>−1 inclusive.




For example, a line:

2 3 2 1 1

will create the following table configuration:

<table width="332">

 <tbody>

  <tr>

   <td width="111">Table Size</td>

   <td width="221">Table IDs</td>

  </tr>

  <tr>

   <td width="111">1</td>

   <td width="221">0, 1</td>

  </tr>

  <tr>

   <td width="111">2</td>

   <td width="221">2, 3, 4</td>

  </tr>

  <tr>

   <td width="111">3</td>

   <td width="221">5, 6</td>

  </tr>

  <tr>

   <td width="111">4</td>

   <td width="221">7</td>

  </tr>

  <tr>

   <td width="111">5</td>

   <td width="221">8</td>

  </tr>

 </tbody>

</table>







Subsequent lines are in one of the following forms:

<ul>

 <li>Enter <em>&lt;id&gt;</em> <em>&lt;num_people&gt;</em> – this is a command indicating that a group with the given <em>id</em> and number of people <em>num_people</em> have arrived at your restaurant.</li>

 <li>Leave <em>&lt;id&gt;</em> – this is a command indicating that the group with the given <em>id</em> just vacated their assigned table; this command will only be issued if the specified group is already seated at a table</li>

 <li>. – a literal period indicates a synchronisation point for the driver (see below for details)</li>

</ul>




Note that the strings “Enter” and “Leave” must be capitalised exactly as shown.




You can make the following <strong>assumptions </strong>about the input we will use for grading:

<ul>

 <li>It is guaranteed that at the end of the input, the queue will be empty and there are no groups remaining in the restaurant.</li>

 <li>All group ids are unique within one simulation run.</li>

 <li>For each “Enter” command, 1 ≤ <em>num_people</em> ≤ 5. – The input file is less than 1000 lines long.</li>

</ul>




Notice that the group id supplied in the input file is never given to your synchronisation mechanism.  This is by design – the behaviour of your code should not depend on the group id.




<h3>Parallelism in the driver</h3>




Like in the previous section, input commands are separated by synchronisation points (“.”). The driver batches together all commands until it encounters a synchronisation point or EOF, and then it executes those commands simultaneously in parallel (each group gets its own thread).  The driver is designed to execute those commands at maximum concurrency.

<strong> </strong>

<h3>Driver output format</h3>




Under normal operation, four types of log messages are printed by the driver:

<ul>

 <li>Group &lt;id&gt; with &lt;num_people&gt; people arrived – this message is printed immediately before request_for_table() is called</li>

 <li>Group &lt;id&gt; is enqueued – this message is printed after on_enqueue() is called, and should appear after the arrival messages, without the need to enter any further commands. Note that this message is always printed, even if the group is not blocked in the queue (as specified earlier, you should always call on_enqueue() in request_for_table()).</li>

 <li>Group &lt;id&gt; is seated at table &lt;table_id&gt; – this message is printed when request_for_table()</li>

 <li>Group &lt;id&gt; left – this message is printed after leave_table()</li>

</ul>




<h3>Driver behaviour overview</h3>




Upon receiving a batch of <em>Enter</em> or <em>Leave</em> commands, the driver does the following (in order):

<ul>

 <li>For every <em>Enter</em> command, prints the <em>group arrived</em> log message.</li>

 <li>For every <em>Enter</em> command, spawns a new thread and calls request_for_table(); for every <em>Leave</em> command, finds the associated thread and calls leave_table().</li>

 <li>Sleeps for 100ms.</li>

 <li>Waits until the on_enqueue() function is called the same number of times as which request_for_table() was called (i.e., waits for all arriving groups in the batch to be either assigned a table or be blocked in the queue).</li>

 <li>For every <em>Enter</em> command, checks that on_enqueue() was called, and prints the <em>group enqueued</em> log message; for every <em>Leave</em> command, waits for leave_table() to complete, joins the thread, and prints the <em>group left</em> log message</li>

 <li>For every thread that newly returned from their call to request_for_table(), prints the <em>group seated</em> log message.</li>

</ul>




All log messages are printed from the main thread (i.e., the thread controlled solely by the driver and not used by any group). The driver implements the necessary synchronisation mechanisms to communicate the events between the group threads and the main thread for printing.




Due to the above flow, all <em>group arrived</em> log messages will be printed before all <em>group enqueued</em> and <em>group left</em> log messages, and those in turn will be printed before all <em>group seated</em> log messages.




For grading purposes, our grader that replaces this driver will figure out (through other means) the groups that should get assigned a table by the end of each batch of commands, and waits for all of them to return from their call to request_for_table() before proceeding. This guarantees that even if the system is slow and 100ms is not enough time for all groups to react to the incoming events, we will still be able to grade your solution correctly.




<h3>Interactivity</h3>




Note that test cases need to be adaptive – a later command may depend on the output of previous commands. Specifically, you can only issue a <em>Leave g </em>command, if group <em>g</em> has already been assigned a table (i.e., <em>Group g is seated at table t </em>has been printed).  This means that some of the commands of our sample test cases may be invalid depending on your output (i.e., depending on how you order concurrently arriving groups in the queue), and you will need to type in the correct commands <strong>depending on the output that has been printed</strong>.  The parallel test case for exercise 4 has comments that will make this clear.




During grading, we will use an adaptive grader that decides on the commands to issue based on the log messages that it has seen previously.  <strong>Hence, you will be able to assume that every command issued by our grader is valid.</strong>







<h2>3.2. Exercise 4</h2>




In this exercise, <strong>each table may only be assigned to a group with the same number of people as seats at the table</strong>.  For example, a group with 3 people may only be assigned to a table with 3 seats.




The following example will help to illustrate this rule.




<h3>Example test case (ex4/seq_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Input                                                                                    Possible output</td>

  </tr>

  <tr>

   <td width="553">2 3 2 1 1Enter 101 3 .Group 101 with 3 people arrivedGroup 101 is enqueuedGroup 101 is seated at table 5Enter 102 3 .Group 102 with 3 people arrivedGroup 102 is enqueuedGroup 102 is seated at table 6Enter 103 3 .(At this point, all tables with 3 seats are occupied, so group 103 has to wait in the queue)Group 103 with 3 people arrivedGroup 103 is enqueuedEnter 104 4 .(Even though group 103 is in front of group 104, group 103 cannot currently be assigned a table, sogroup 104 gets to jump the queue and be assigned a table immediately)Group 104 with 4 people arrivedGroup 104 is enqueuedGroup 104 is seated at table 7Enter 105 3 .Group 105 with 3 people arrivedGroup 105 is enqueuedEnter 106 4 .Group 106 with 4 people arrivedGroup 106 is enqueuedLeave 102 .(As group 102 leaves, their table with 3 seats becomes vacated, so group 103 takes that table)Group 102 leftGroup 103 is seated at table 6Leave 101 .Group 101 left</td>

  </tr>

  <tr>

   <td width="553">Group 105 is seated at table 5Enter 107 1 .Group 107 with 1 people arrivedGroup 107 is enqueuedGroup 107 is seated at table 0Leave 104 .Group 104 leftGroup 106 is seated at table 7Leave 103 .(The table remains vacated after group 103 leaves, since there are no groups with 3 people in thequeue)Group 103 leftLeave 106 .Group 106 leftLeave 107 .Group 107 leftLeave 105 .Group 105 leftEnter 108 3 .Group 108 with 3 people arrivedGroup 108 is enqueuedGroup 108 is seated at table 5Enter 109 3 .Group 109 with 3 people arrivedGroup 109 is enqueuedGroup 109 is seated at table 6Enter 110 3 .Group 110 with 3 people arrivedGroup 110 is enqueuedLeave 109 .Group 109 leftGroup 110 is seated at table 6Leave 110 .Group 110 leftLeave 108(Ctrl+D pressed to end input stream)Group 108 left</td>

  </tr>

 </tbody>

</table>




<strong>           </strong>

<h3>Example test case (ex4/par_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Input                                                                                    Possible output</td>

  </tr>

  <tr>

   <td width="553">1 2 3 2 1Enter 150 2Enter 185 3 .(Note that the order within each pair of the three pairs of messages below do not matter)Group 185 with 3 people arrivedGroup 150 with 2 people arrivedGroup 185 is enqueuedGroup 150 is enqueuedGroup 150 is seated at table 1Group 185 is seated at table 3(At this point, the queue contains no groups)Enter 367 2Enter 374 2 .(As there is only one remaining available table with 2 seats, only one of the newly arrived groups can immediately sit at a table.  As both groups arrived at the same time, either group may be placed before the other in the queue. The group that is placed before the other – in this case group 367 –should be seated immediately.)Group 374 with 2 people arrivedGroup 367 with 2 people arrivedGroup 374 is enqueuedGroup 367 is enqueuedGroup 367 is seated at table 2(At this point, the queue contains just group 374)Enter 776 2Leave 150Enter 777 2 .(When group 150 leaves, group 374 will take the vacated table, since it arrived earlier than groups 776 and 777)Group 777 with 2 people arrivedGroup 776 with 2 people arrivedGroup 777 is enqueuedGroup 150 leftGroup 776 is enqueuedGroup 374 is seated at table 1(At this point, the queue contains groups 776 and 777, in an order known only to your implementation (but unknown to the reader of this output log))Leave 367Enter 420 5 .(When group 367 leaves, either group 776 or 777, whichever is closer to the front of the queue, will take the vacated table; in this case group 776 is the group that is seated)(Note that group 420 is seated even though group 777 is still in the queue and is closer to the front ofthe queue, since the table with 5 seats can only be assigned to group 420)Group 420 with 5 people arrivedGroup 420 is enqueuedGroup 367 left Group 776 is seated at table 2</td>

  </tr>

 </tbody>

</table>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Group 420 is seated at table 8(At this point, the queue contains just group 777)Leave 374 .Group 374 leftGroup 777 is seated at table 1(At this point, the queue contains no groups)Enter 418 5Leave 777 .Group 418 with 5 people arrived Group 777 leftGroup 418 is enqueued(At this point, the queue contains just group 418)Leave 776Leave 420Leave 185Enter 143 1Enter 1523 1 .(The three “leave” commands will make the restaurant completely empty.  Group 418, and either one of group 143 or 1523, will be seated.  In this test run, group 143 happens to be enqueued first, andhence get assigned the sole table with 1 seat.)Group 1523 with 1 people arrivedGroup 143 with 1 people arrivedGroup 1523 is enqueuedGroup 143 is enqueuedGroup 185 leftGroup 420 leftGroup 776 left Group 418 is seated at table 8Group 143 is seated at table 0(At this point, the queue contains just group 1523)Leave 418 .Group 418 left(At this point, the queue contains just group 1523, and the restaurant only has one table occupied (by group 143))Leave 143Enter 144 1Enter 111 1Enter 419 1 .(The table vacated by group 143 must be assigned to group 1523, since it queued up before the 3groups that just arrived.)Group 419 with 1 people arrivedGroup 111 with 1 people arrivedGroup 144 with 1 people arrivedGroup 419 is enqueuedGroup 111 is enqueuedGroup 144 is enqueuedGroup 143 left</td>

  </tr>

  <tr>

   <td width="553">Group 1523 is seated at table 0(At this point, the queue contains groups 144, 111, and 419, in an order known only to your implementation)Leave 1523 .(It turns out that group 111 was enqueued before groups 144 and 418, so group 111 gets the tablefirst)Group 1523 leftGroup 111 is seated at table 0(At this point, the queue contains groups 144 and 419, in an order known only to your implementation)Leave 111 .(It turns out that group 419 was enqueued before group 144)Group 111 leftGroup 419 is seated at table 0(At this point, the queue contains just group 144)Leave 419 .Group 419 leftGroup 144 is seated at table 0(At this point, the queue contains no groups)Leave 144(Ctrl+D pressed to end input stream)Group 144 left</td>

  </tr>

 </tbody>

</table>




Note that the last few commands may have to be modified depending on the actual order groups 144, 111, and 419 go into the queue, to ensure that we don’t instruct a group that is not already seated to leave.




For example, after Group 1523 left, if Group 419 gets table 0 first (because it was enqueued earlier) and the driver prints “Group 419 is seated at table 0” first, then the valid next input would be “Leave 419” (“Leave 111” would no longer be valid, because Group 111 has not been assigned a table yet).




<h2>3.3. Exercise 5</h2>




In this exercise, <strong>each group may be assigned to a table that has at least as many seats as people in that group</strong>. For example, a group with 3 people may be assigned to a table with 3, 4, or 5 seats.




However, if there are multiple available tables that a group can be assigned to, you should assign the group to a table with <strong>minimum number of extra seats</strong>. And if there are multiple such tables, you may pick any one of them. For example, suppose a group with 3 people arrives and there are two empty tables, one with 4 seats and one with 5 seats. You should assign the table with 4 seats to the group.




The given test inputs for exercise 4 are also valid for exercise 5, but the output will differ.




<h3>Example test case (ex4/seq_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Input                                                                                    Possible output</td>

  </tr>

  <tr>

   <td width="553">2 3 2 1 1Enter 101 3 .Group 101 with 3 people arrivedGroup 101 is enqueuedGroup 101 is seated at table 5Enter 102 3 .Group 102 with 3 people arrivedGroup 102 is enqueuedGroup 102 is seated at table 6Enter 103 3 .(At this point, all tables with 3 seats are occupied, so group 103 gets assigned to a table with 4 seats)Group 103 with 3 people arrivedGroup 103 is enqueuedGroup 103 is seated at table 7Enter 104 4 .(Since all tables with 4 seats are occupied, group 104 gets assigned to a table with 5 seats)Group 104 with 4 people arrivedGroup 104 is enqueuedGroup 104 is seated at table 8Enter 105 3 .(There are no empty tables with at least 3 seats, so group 105 waits in the queue)Group 105 with 3 people arrivedGroup 105 is enqueuedEnter 106 4 .Group 106 with 4 people arrived</td>

  </tr>

 </tbody>

</table>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Group 106 is enqueuedLeave 102 .(As group 102 leaves, their table with 3 seats becomes vacated; group 103 is at the front of thequeue and hence takes that table)Group 102 leftGroup 105 is seated at table 6Leave 101 .(As group 104 leaves, their table with 3 seats becomes vacated; but there are no groups with at most 3 people in the queue, so the table remains empty)Group 101 leftEnter 107 1 .Group 107 with 1 people arrivedGroup 107 is enqueuedGroup 107 is seated at table 0Leave 104 .(As group 104 leaves, their table with 5 seats becomes vacated; group 106 is the only group in thequeue at this point, and they can sit at this table, so they are assigned to it)Group 104 leftGroup 106 is seated at table 8Leave 103 .(The table remains vacated after group 103 leaves, since there are no groups with at most 4 table inthe queue)Group 103 leftLeave 106 .Group 106 leftLeave 107 .Group 107 leftLeave 105 .Group 105 leftEnter 108 3 .(Group 108 can be assigned to either table 5 or 6; in this example, they choose table 6)Group 108 with 3 people arrivedGroup 108 is enqueuedGroup 108 is seated at table 6Enter 109 3 .Group 109 with 3 people arrivedGroup 109 is enqueuedGroup 109 is seated at table 5Enter 110 3 .</td>

  </tr>

  <tr>

   <td width="553">(Note that group 110 must take table 7 – there are no empty tables with 3 seats.  Group 110 cannot take table 8 because they must choose an available table with the minimum number of remainingempty seats.)Group 110 with 3 people arrivedGroup 110 is enqueuedGroup 110 is seated at table 7Leave 109 .Group 109 leftLeave 110 .Group 110 leftLeave 108(Ctrl+D pressed to end input stream)Group 108 left</td>

  </tr>

 </tbody>

</table>




The following is a test to check the correctness of your synchronisation mechanism under heavy load.




<h3>Example test case (ex5/load_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="273">Input</td>

   <td width="280">Possible output</td>

  </tr>

  <tr>

   <td width="273">2 2 2 2 2Enter 201 1Enter 202 1Enter 203 1Enter 204 1Enter 205 1Enter 206 1Enter 207 1Enter 208 1Enter 209 1Enter 210 1Enter 211 1Enter 212 1 .</td>

   <td width="280">Group 212 with 1 people arrivedGroup 211 with 1 people arrivedGroup 210 with 1 people arrivedGroup 209 with 1 people arrivedGroup 208 with 1 people arrivedGroup 207 with 1 people arrivedGroup 206 with 1 people arrivedGroup 205 with 1 people arrivedGroup 204 with 1 people arrivedGroup 203 with 1 people arrivedGroup 202 with 1 people arrivedGroup 201 with 1 people arrivedGroup 212 is enqueuedGroup 211 is enqueued</td>

  </tr>

 </tbody>

</table>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Group 210 is enqueuedGroup 209 is enqueuedGroup 208 is enqueuedGroup 207 is enqueuedGroup 206 is enqueuedGroup 205 is enqueuedGroup 204 is enqueuedGroup 203 is enqueuedGroup 202 is enqueuedGroup 201 is enqueuedGroup 209 is seated at table 9Group 203 is seated at table 1Group 208 is seated at table 3Group 202 is seated at table 2Group 210 is seated at table 8Group 201 is seated at table 7Group 207 is seated at table 6Group 211 is seated at table 5Group 205 is seated at table 4Group 204 is seated at table 0Enter 213 2Enter 214 3 .Group 214 with 3 people arrivedGroup 213 with 2 people arrivedGroup 214 is enqueuedGroup 213 is enqueuedLeave 205 .(Note that group 206 or 212 must take the vacated table (that has 3 seats) even though there aresufficient seats for groups 213 and 214, since groups 206 and 212 arrived earlier)Group 205 leftGroup 212 is seated at table 4Leave 212 .(Similarly, group 206 must take the vacated table)Group 212 leftGroup 206 is seated at table 4Leave 206 .(Now, either group 213 or 214 (whichever arrived first) should take the vacated table)Group 206 leftGroup 213 is seated at table 4Leave 213 .Group 213 leftGroup 214 is seated at table 4Leave 214 .(There are no more groups in the queue, so the table vacated by group 214 remains empty)</td>

  </tr>

  <tr>

   <td width="553">Group 214 leftLeave 201Leave 202Leave 203Leave 204Leave 207Leave 208Leave 209Leave 210Leave 211(Ctrl+D pressed to end input stream)Group 211 leftGroup 210 leftGroup 209 leftGroup 208 leftGroup 207 leftGroup 204 leftGroup 203 leftGroup 202 leftGroup 201 left</td>

  </tr>

 </tbody>

</table>




You are also recommended to modify ex4/par_test adaptively, and create additional test cases to verify that your synchronisation mechanism is correct.







<h2>3.4. Exercise 6</h2>




This is a bonus exercise.




The restaurant now wants to admit as many groups as possible, and so will now <strong>allow multiple groups to sit at the same table</strong>. However, groups prefer not to share a table with other groups <em>unless </em>there are no other empty tables.




In this exercise, <strong>each group may be assigned to a table that has at least as many <u>empty</u> seats as people in that group</strong>. For example, a group with 3 people may be assigned to a table with 3, 4, or 5 empty seats. The table itself may have other non-empty seats that are used concurrently by another group.




However, if there are multiple possible tables that a group can be assigned to, you should do the following:

<ul>

 <li>If there are tables that are totally empty and have at least as many seats as people in the group, assign the group to such a table with <strong>minimum number of extra seats</strong>. (If there are multiple such tables, you may pick any one of them.)</li>

 <li>Otherwise, if there are tables that are partially occupied and have at least as many empty seats as people in the group, assign the group to such a table with the <strong>minimum number of empty seats</strong>. (If there are multiple such tables, you may pick any one of them.)</li>

 <li>Otherwise, keep this group in the queue.</li>

</ul>




The following example interaction will help to illustrate this rule.




<h3>Example test case (ex6/share_test.in)</h3>




<table width="553">

 <tbody>

  <tr>

   <td width="240">Input</td>

   <td width="313">Possible output</td>

  </tr>

  <tr>

   <td width="240">1 2 1 1 1Enter 601 1 .</td>

   <td width="313">Group 601 with 1 people arrivedGroup 601 is enqueuedGroup 601 is seated at table 0</td>

  </tr>

  <tr>

   <td width="240">Enter 602 1Enter 603 1Enter 604 1Enter 605 1 .</td>

   <td width="313">(These four arriving groups must seat on tables 1, 2, 3, 4)Group 605 with 1 people arrivedGroup 604 with 1 people arrivedGroup 603 with 1 people arrivedGroup 602 with 1 people arrivedGroup 605 is enqueuedGroup 604 is enqueuedGroup 603 is enqueuedGroup 602 is enqueued</td>

  </tr>

 </tbody>

</table>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Group 605 is seated at table 4Group 604 is seated at table 1Group 602 is seated at table 3Group 603 is seated at table 2Enter 606 1 .(Group 606 must seat on table 5)Group 606 with 1 people arrivedGroup 606 is enqueuedGroup 606 is seated at table 5Enter 607 1 .(As there are no empty tables, group 607 must share a table with someone else.  They must sit attable 1 or 2, since those two tables have only one empty seat each)Group 607 with 1 people arrivedGroup 607 is enqueuedGroup 607 is seated at table 1Enter 608 1Enter 609 1Enter 610 1 .(The three arriving groups must sit at the one empty seat at table 2 and the two empty seats at table 3)Group 610 with 1 people arrivedGroup 609 with 1 people arrivedGroup 608 with 1 people arrivedGroup 610 is enqueuedGroup 609 is enqueuedGroup 608 is enqueuedGroup 608 is seated at table 2Group 610 is seated at table 3Group 609 is seated at table 3Enter 611 1Enter 612 1Enter 613 1 .(The three arriving groups must sit at the three empty seats at table 4)Group 613 with 1 people arrivedGroup 612 with 1 people arrivedGroup 611 with 1 people arrivedGroup 613 is enqueuedGroup 612 is enqueuedGroup 611 is enqueuedGroup 613 is seated at table 4Group 612 is seated at table 4Group 611 is seated at table 4Leave 611Leave 602Leave 609 .Group 609 left</td>

  </tr>

 </tbody>

</table>




<table width="553">

 <tbody>

  <tr>

   <td width="553">Group 602 leftGroup 611 leftEnter 614 1 .(Group 614 must sit at table 4, since all tables are not totally empty and table 4 is the only table with exactly one empty seat.  Note that even though table 3 has a smaller <em>total</em> number of seats than table 4, they must sit at table 4 because table 4 has a smaller number of <em>empty</em> seats than table 3.)Group 614 with 1 people arrivedGroup 614 is enqueuedGroup 614 is seated at table 4Enter 615 3 .Group 615 with 3 people arrivedGroup 615 is enqueuedGroup 615 is seated at table 5Enter 616 3 .Group 616 with 3 people arrivedGroup 616 is enqueuedEnter 617 2 .Group 617 with 2 people arrivedGroup 617 is enqueuedGroup 617 is seated at table 3Leave 612Leave 613 .(After groups 612 and 613 leave, there are two empty spaces at table 4; however, group 616 has 3people so they must still remain in the queue)Group 613 leftGroup 612 leftLeave 614 .(After group 614 leaves, there are three empty spaces at table 4, so group 616 is assigned to thattable (which is shared with group 605))Group 614 leftGroup 616 is seated at table 4Leave 601 .Group 601 leftEnter 618 2 .Group 618 with 2 people arrivedGroup 618 is enqueuedLeave 604Leave 607 .(After groups 604 and 607 leave, table 1 (with 2 seats) becomes totally empty, so group 618 isassigned to table 1)Group 607 leftGroup 604 left</td>

  </tr>

  <tr>

   <td width="553">Group 618 is seated at table 1Enter 619 1Enter 620 1Enter 621 2Enter 622 1 .Group 619 with 1 people arrivedGroup 620 with 1 people arrivedGroup 621 with 2 people arrivedGroup 622 with 1 people arrivedGroup 619 is enqueuedGroup 620 is enqueuedGroup 621 is enqueuedGroup 622 is enqueuedGroup 619 is seated at table 0Group 622 is seated at table 5Leave 615 .(As group 615 leaves, 3 seats are vacated, and both groups 620 and 621 can take those seats)Group 615 left Group 620 is seated at table 5Group 621 is seated at table 5Leave 619Leave 620Leave 621Leave 603Leave 608Leave 610Leave 617Leave 605Leave 616Leave 606Leave 618Leave 622(Ctrl+D pressed to end input stream)Group 622 leftGroup 618 leftGroup 606 leftGroup 616 leftGroup 605 leftGroup 617 leftGroup 610 leftGroup 608 leftGroup 603 leftGroup 621 leftGroup 620 leftGroup 619 left</td>

  </tr>

 </tbody>

</table>




You are also recommended to test out your implementation with the test cases from exercises 4 and 5.