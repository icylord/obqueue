# obqueue
obqueue is a awesome fast/simple queue

# test_obqueue_

<pre><code>
#include "obqueue.h"
#include <pthread.h>


#define DUMMY_VALUE	2341321

long COUNTS_PER_THREAD = 2500000;
obqueue_t qq;
void* producer(void* index) {
	obqueue_t* q = &qq;		
	handle_t* th = (handle_t*) malloc(sizeof(handle_t));
	memset(th, 0, sizeof(handle_t));
	queue_register(q, th, ENQ);
	
	for(;; ) {
		pthread_barrier_wait(&pro_barrier);
		for(int i = 0; i < COUNTS_PER_THREAD; ++i)  
			enqueue(q, th, DUMMY_VALUE);
		pthread_barrier_wait(&pro_barrier);
	}	
	return NULL;
}

#define THREAD_NUM 2
int* ints;
void* consumer(void* index) {

	obqueue_t* q = &qq;
	handle_t* th = (handle_t*) malloc(sizeof(handle_t));
	memset(th, 0, sizeof(handle_t));
	queue_register(q, th, DEQ);
	
	for(;;) {
		pthread_barrier_wait(&con_barrier);	
		for(long i = 0; i < COUNTS_PER_THREAD; ++i)  {
			int value;
			if((value = dequeue(q, th)) != DUMMY_VALUE) {
				printf("IS WRONG!!!!!!\n");
				exit(0);
			}
		}
		pthread_barrier_wait(&con_barrier);
	}
	
	fflush(stdout);
	return NULL;
}

int main(int argc, char* argv[]) {

	printf("thread number: %d\n", THREAD_NUM);
	
	pthread_barrier_init(&pro_barrier, NULL, THREAD_NUM + 1);
	pthread_barrier_init(&con_barrier, NULL, THREAD_NUM + 1);
	printf("cpu: %d\n", get_nprocs_conf());
	if(argc >= 3) {
		COUNTS_PER_THREAD = atol(argv[1]);
		threshold = atoi(argv[2]);	
	}
	
	printf("here %ld\n", THREAD_NUM * COUNTS_PER_THREAD);
	fflush(stdout);
	init_queue(&qq, THREAD_NUM, THREAD_NUM);
	
	pthread_t pids[THREAD_NUM];
	
	for(int i = 0; i < THREAD_NUM; ++i) {
		if(-1 == pthread_create(&pids[i], NULL, producer, i)) {
			printf("error create thread\n");
			exit(1);
		}
		if(-1 == pthread_create(&pids[i], NULL, consumer, i)) {
			printf("error create thread\n");
			exit(1);
		}
	}
	
	for(int i = 0; i < 8;) {
	
		printf("\n%d times\n", i);
		
		pthread_barrier_wait(&con_barrier);	
		sleep(1);
		struct timeval start;
		gettimeofday(&start, NULL);
		pthread_barrier_wait(&pro_barrier);
		
		pthread_barrier_wait(&pro_barrier);
		pthread_barrier_wait(&con_barrier);	

		
		struct timeval pro_end;
		gettimeofday(&pro_end, NULL);
		float cost_time = (pro_end.tv_sec-start.tv_sec)+(pro_end.tv_usec-start.tv_usec) / 1000000.0;
		printf("pro cost times: %f\n", cost_time);
		printf("%d times over\n", i);
		fflush(stdout);
		++i;
	}
	return 0;
}
</pre></code>
