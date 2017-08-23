# CSE421-OS-Pintos
Operating System Projects


+--------------------+
			|        CS 140      |
			| PROJECT 1: THREADS |
			|   DESIGN DOCUMENT  |
			+--------------------+
				   
---- GROUP ----

>> Fill in the names and email addresses of your group members.

Yonghyun Park <yonghyun@buffalo.edu>


---- PRELIMINARIES ----

>> If you have any preliminary comments on your submission, notes for the
>> TAs, or extra credit, please give them here.

>> Please cite any offline or online sources you consulted while
>> preparing your submission, other than the Pintos documentation, course
>> text, lecture notes, and course staff.

			     ALARM CLOCK
			     ===========

---- DATA STRUCTURES ----

>> A1: Copy here the declaration of each new or changed `struct' or
>> `struct' member, global or static variable, `typedef', or
>> enumeration.  Identify the purpose of each in 25 words or less.

/*sleep_thread*/

Sleep_ thread is a struct to save ticks to wake up and semaphore. Since I want to use List functions implemented on pintos, I also added List_elem.

Struct sleep_thread{

Int64_t wake_up_ticks;

Struct semaphore sema;

Struct List_elem elem;

}

/*sleeping list*/

	This is a collections of sleep_thread(struct). When timer sleep is called, the current thread goes to here.

Struct list sleeping_list;

---- ALGORITHMS ----

>> A2: Briefly describe what happens in a call to timer_sleep(),
>> including the effects of the timer interrupt handler.

	Timer_ticks() is a hardware interrupt, so it keeps going. Whenever timer sleep is called, it timer_sleep will save ticks to be woke up in sleep_thread(I implemented).After that, it initializes a semaphore for communication. (the sem_init saves the information about current thread) eventually the sleep_thread goes to sleeping_list by ticks( to use the insert_ordered, implementation of compared function is needed. After that is done, it sema_down. Additionally, when we sleep the threads, we do not have to worry about blocking the threads, beacsue sema_down takes care of it( it is implemented on sema_down)


>> A3: What steps are taken to minimize the amount of time spent in
>> the timer interrupt handler?

	When it is the time to wake up for the sleeping threads, a function is needed to wake up a threads form the sleeping_list. While the sleeping_list is not empty, we can use list_pop_front. Since the sleep_thread elems in the list are sorted by smallest to largest ticks, it will automatically return a thread is soonest and if the current ticks are already passed the wake_up_ticks, it sema_up, so that the real thread has the ticks can go to ready list. If the time did not pass, we put it back to the sleeping_list. This can happen due to the timer_interupt(). So the waking up fuction should be implemented in timer_interupt(), so that the waking up function is able to detect when to wake up threads

---- SYNCHRONIZATION ----

>> A4: How are race conditions avoided when multiple threads call
>> timer_sleep() simultaneously?

	In the data structure we designed, there is a semaphore. So even though many threads call timer_sleep(), the semaphore in the sleep_thread will take care of the situation.

>> A5: How are race conditions avoided when a timer interrupt occurs
>> during a call to timer_sleep()?

To be honest, we just disabled it.

	

---- RATIONALE ----

>> A6: Why did you choose this design?  In what ways is it superior to
>> another design you considered?
	Actually, we could just save the wake_up_ticks in the auctual thread struct. However, since struct thread is a data many functions access, we believe that we rather want to make another struct has semaphore and wake_up_ticks. Semaphore also has a waiter_list. So the point of our implementation is to avoid crowd places.

			 PRIORITY SCHEDULING
			 ===================

---- DATA STRUCTURES ----

>> B1: Copy here the declaration of each new or changed `struct' or
>> `struct' member, global or static variable, `typedef', or
>> enumeration.  Identify the purpose of each in 25 words or less.

>> B2: Explain the data structure used to track priority donation.
>> Use ASCII art to diagram a nested donation.  (Alternately, submit a
>> .png file.)

---- ALGORITHMS ----

>> B3: How do you ensure that the highest priority thread waiting for
>> a lock, semaphore, or condition variable wakes up first?

>> B4: Describe the sequence of events when a call to lock_acquire()
>> causes a priority donation.  How is nested donation handled?

>> B5: Describe the sequence of events when lock_release() is called
>> on a lock that a higher-priority thread is waiting for.

---- SYNCHRONIZATION ----

>> B6: Describe a potential race in thread_set_priority() and explain
>> how your implementation avoids it.  Can you use a lock to avoid
>> this race?

---- RATIONALE ----

>> B7: Why did you choose this design?  In what ways is it superior to
>> another design you considered?

			  ADVANCED SCHEDULER
			  ==================

---- DATA STRUCTURES ----

>> C1: Copy here the declaration of each new or changed `struct' or
>> `struct' member, global or static variable, `typedef', or
>> enumeration.  Identify the purpose of each in 25 words or less.

In struct thread, in thread.h

int nice;                            Nice value of the thread. 
int32_t recent_cpu;                  17.14 Fixed-Point representation. 

In thread.c

int32_t load_avg                this is to save the load_avg

---- ALGORITHMS ----

>> C2: Suppose threads A, B, and C have nice values 0, 1, and 2.  Each
>> has a recent_cpu value of 0.  Fill in the table below showing the
>> scheduling decision and the priority and recent_cpu values for each
>> thread after each given number of timer ticks:

timer  recent_cpu    priority   thread
ticks   A   B   C   A   B   C   to run
-----  --  --  --  --  --  --   ------
 0      0   0   0  63  61  58     A
 4      4   0   0  62  61  59     A
 8      8   0   0  61  61  59     B
12      8   4   0  61  60  59     A
16     12   4   0  60  60  59     B
20     12   8   0  60  59  59     A
24     16   8   0  59  59  59     C
28     16  12   0  59  59  58     B
32     16  12   4  59  58  58     A
36     20  12   4  58  58  58     C

>> C3: Did any ambiguities in the scheduler specification make values
>> in the table uncertain?  If so, what rule did you use to resolve
>> them?  Does this match the behavior of your scheduler?

Actually, we are still working on this. However,the things I expect to be ambiguous comes from clock tick. Since the recent_cpu is mainly decided by the clock tick, even small a small mistake of calculating time would result different scheduling.

>> C4: How is the way you divided the cost of scheduling between code
>> inside and outside interrupt context likely to affect performance?

Since recalculating has to be done before the schedule is called, getting correct time is important. So I think that if it does not get accurate time, the next_thread_to_run() would pick a wrong thread.

---- RATIONALE ----

>> C5: Briefly critique your design, pointing out advantages and
>> disadvantages in your design choices.  If you were to have extra
>> time to work on this part of the project, how might you choose to
>> refine or improve your design?

Acturally, the thing was it was really tempting that using hardware interrupts, because it is more simple than using semaphores. At some points, I was not able to implement functions with synchronization primitives. If I were given more time, I would work on synchronization primitives.

>> C6: The assignment explains arithmetic for fixed-point math in
>> detail, but it leaves it open to you to implement it.  Why did you
>> decide to implement it the way you did?  If you created an
>> abstraction layer for fixed-point math, that is, an abstract data
>> type and/or a set of functions or macros to manipulate fixed-point
>> numbers, why did you do so?  If not, why not?

To be honest my fixed-point function is not working perfect, but I just followed the direction provided in the Pintos manual appendix. I also think that using macro is simpler.

			   SURVEY QUESTIONS
			   ================

Answering these questions is optional, but it will help us improve the
course in future quarters.  Feel free to tell us anything you
want--these questions are just to spur your thoughts.  You may also
choose to respond anonymously in the course evaluations at the end of
the quarter.

>> In your opinion, was this assignment, or any one of the three problems
>> in it, too easy or too hard?  Did it take too long or too little time?
Alarm-clock was easiest one. It also took a lot of time


>> Did you find that working on a particular part of the assignment gave
>> you greater insight into some aspect of OS design?

I had more understanding of hard ware. Since I am majoring computer science, I did not have many chances to consider hardware.

>> Is there some particular fact or hint we should give students in
>> future quarters to help them solve the problems?  Conversely, did you
>> find any of our guidance to be misleading?

It took long time to figure out how to use semaphores,so I recommend other students understand what is actually going on from lectures

>> Do you have any suggestions for the TAs to more effectively assist
>> students, either for future quarters or the remaining projects?

I think TAs could go over more example of stuffs we learn in lectures.
