# traffic-intersection-problem

#include <stdlib.h>
#include <stdio.h>
#include <pthread.h>

typedef enum {
LEFT=0,
RIGHT,
STRAIGHT
} turn_direction_t;

typedef enum {
NORTH=0,
SOUTH,
EAST,
WEST
} orientation_t;

static void gostraight(orientation_t, unsigned int);
static void turnleft(orientation_t, unsigned int);
static void turnright(orientation_t, unsigned int);
static void * approachintersection(void*);
int createcars(int, char**);


pthread_mutex_t NW_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t NE_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t SE_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t SW_mutex = PTHREAD_MUTEX_INITIALIZER;


void lock(CAR, MUTEX) {do 
{         
if (pthread_mutex_lock(&MUTEX)) { 
  fprintf(stderr, "pthread_mutex_lock error %d\n", CAR);    
  exit(-1); 
} 
} while(0);}

void unlock(CAR, MUTEX) 
{do 
{     
 if (pthread_mutex_unlock(&MUTEX)) { 
  fprintf(stderr, "pthread_mutex_unlock error %d\n", CAR);  
  exit(-1); 
} 
} while(0);}



#define NUMCARS 30

#define lock_NW(CAR) lock(CAR, NW_mutex)
#define lock_NE(CAR) lock(CAR, NE_mutex)
#define lock_SW(CAR) lock(CAR, SW_mutex)
#define lock_SE(CAR) lock(CAR, SE_mutex)

#define unlock_NW(CAR) unlock(CAR, NW_mutex)
#define unlock_NE(CAR) unlock(CAR, NE_mutex)
#define unlock_SW(CAR) unlock(CAR, SW_mutex)
#define unlock_SE(CAR) unlock(CAR, SE_mutex)

static void gostraight(orientation_t cardirection, unsigned int carnumber){

switch(cardirection){
case NORTH:
lock_NE(carnumber);
lock_SE(carnumber);
printf("Car %d, Moving South-North\n", carnumber);
unlock_SE(carnumber);
unlock_NE(carnumber);
break;
case SOUTH:
lock_NW(carnumber);
lock_SW(carnumber);
printf("Car %d, Moving North-South\n", carnumber);
unlock_SW(carnumber);
unlock_NW(carnumber);
break;
case EAST:
lock_SE(carnumber);
lock_SW(carnumber);
printf("Car %d, Moving West-East\n", carnumber);
unlock_SW(carnumber);
unlock_SE(carnumber);
break;
case WEST:
lock_NW(carnumber);
lock_NE(carnumber);
printf("Car %d, Moving East-West\n", carnumber);
unlock_NE(carnumber);
unlock_NW(carnumber);
break;
}
}

static void turnleft(orientation_t cardirection, unsigned int carnumber){

switch(cardirection){
case NORTH:
lock_NE(carnumber);
lock_SE(carnumber);
lock_SW(carnumber);
printf("Car %d, Moving South-West\n", carnumber);
unlock_SW(carnumber);
unlock_SE(carnumber);
unlock_NE(carnumber);
break;
case SOUTH:
lock_NW(carnumber);
lock_NE(carnumber);
lock_SW(carnumber);
printf("Car %d, Moving North-East\n", carnumber);
unlock_SW(carnumber);
unlock_NE(carnumber);
unlock_NW(carnumber);
break;
case EAST:
lock_NW(carnumber);
lock_SE(carnumber);
lock_SW(carnumber);
printf("Car %d, Moving West-North\n", carnumber);
unlock_SW(carnumber);
unlock_SE(carnumber);
unlock_NW(carnumber);
break;
case WEST:
lock_NW(carnumber);
lock_NE(carnumber);
lock_SE(carnumber);
printf("Car %d, Moving East-South\n", carnumber);
unlock_SE(carnumber);
unlock_NE(carnumber);
unlock_NW(carnumber);
break;
}
}

static void turnright(orientation_t cardirection, unsigned int carnumber){

switch(cardirection){
case NORTH:
lock_NE(carnumber);
printf("Car %d, Moving South-East\n", carnumber);
unlock_NE(carnumber);
break;
case SOUTH:
lock_SW(carnumber);
printf("Car %d, Moving North-West\n", carnumber);
unlock_SW(carnumber);
break;
case EAST:
lock_SE(carnumber);
printf("Car %d, Moving West-South\n", carnumber);
unlock_SE(carnumber);
break;
case WEST:
lock_NW(carnumber);
printf("Car %d, Moving East-North\n", carnumber);
unlock_NW(carnumber);
break;
}
}

static void * approachintersection(void* arg){
unsigned int * carnumberptr;
unsigned int carnumber;
orientation_t cardir = (orientation_t)random()%4;
unsigned long turn = random()%3;

carnumberptr = (unsigned int*) arg;
carnumber = (unsigned int) *carnumberptr;

if(turn==LEFT){
turnleft(cardir, carnumber);
} else if(turn==RIGHT){
turnright(cardir, carnumber);
} else {//straight
gostraight(cardir, carnumber);
}

return (void*)carnumberptr;
}

int main(int argc, char **argv){

int index, tid;
unsigned int carids[NUMCARS];
pthread_t carthreads[NUMCARS];

for(index = 0; index <NUMCARS; index++)
{
carids[index] = index;
tid = pthread_create(&carthreads[index], NULL, approachintersection,(void*)&carids[index]);
printf("%d",tid);
}

for(index = 0; index <NUMCARS; index++){
pthread_join(carthreads[index], NULL);
}
printf("Done\n");
return 1;
}
