# custom_block_device

Module_init() 함수로 module 시작                                                                                                      
Module_exit() 함수로 module 종료

register_blkdev() 함수를 이용해 device의 major, minor  number를 할당해준다. (init 함수에서)                                                                 
unregister_blkdev() 함수를 이용해 할당한 번호를 제거한다. (exit 함수에서)

드라이버 전체를 관리할 데이터들을 모아놓을 구조체를 정의해야 한다.
예제에서는 struct mybrd_device 구조체를 선언하였다.
이 구조체에서 가장 중요한 데이터 구조는 struct request_queue와 struct gendisk 두 가지이다.

이후 mybrd_alloc()함수에서 mybrd_device구조체를 채워나간다.                                                                                                                
[단계 1] - 메모리할당                                                                                                                                   
kzalloc을 이용해 객체가 저장될 메모리를 할당한다.                                                                                                                   
spin-lock을 초기화한다.  

[단계 2] - request queue object 생성
request queue에서 빠져나간 request는 mybrd_make_request_fn()함수에서 처리하게 된다.                                                    
blk_alloc_queue_node()함수를 이용해 request queue를 생성한다.                                                                                           
이후 blk_queue_make_request()함수를 이용해 request queue와 mybrd_make_request_fn()함수를 연결해준다.                                                 
*(커널이 request queue를 관리해 주므로 init 부분을 제외하고 추가적으로 신경쓸 부분이 없다. 이 때 모든 초기화 함수들이 커널에서 제공되기때문에 이를 이용)

[단계 3] - gendisk object 생성
