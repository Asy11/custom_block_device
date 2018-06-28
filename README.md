# custom_block_device

Module_init() 함수로 module 시작                                                                                                      
Module_exit() 함수로 module 종료

register_blkdev() 함수를 이용해 device의 major, minor  number를 할당해준다. (init 함수에서)                                                                 
unregister_blkdev() 함수를 이용해 할당한 번호를 제거한다. (exit 함수에서)

드라이버 전체를 관리할 데이터들을 모아놓을 구조체를 정의해야 한다.
예제에서는 struct mybrd_device 구조체를 선언하였다.
이 구조체에서 가장 중요한 데이터 구조는 struct request_queue와 struct gendisk 두 가지이다.

이후 mybrd_alloc()함수에서 mybrd_device구조체를 채워나간다.
[단계 1]
kzalloc을 이용해 객체가 저장될 메모리를 할당한다.
