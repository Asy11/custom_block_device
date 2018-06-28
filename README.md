# custom_block_device

https://github.com/gurugio/book_linuxkernel_blockdrv_ko 를 보고 공부한 내용 정리.

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
gendisk를 생성할 때 전용 할당 함수인 alloc_disk(1)를 이용해 생성한다. 이 때 인자로 준 '1'은 디스크에 최대 파티션 개수를 의미한다.                                                     
디스크 크기는 섹터단위(512Byte)이므로 바이트 단위 숫자를 512로 나눠서 지정한다.
fops필드는 디스크의 장치 파일을 열고 사용할 때 호출되는 함수이다. (struct block_device_operations에 정의되어있음)
*(dump_stack() 이라는 함수가 있는데 이 함수를 콜하면 현재까지의 콜 스택을 커널 메시지로 출력해주는 함수이다. 이를 이용해 디버깅하면 좋다.)

[단계 4] - add disk
add_disk()함수를 통해 앞서 생성한 gendisk를 기반으로 새로운 디스크를 생성한다.                                                
request queue를 생성하긴 했지만 gendisk 객체 없이는 커널에 등록할 수 없다.                                  
즉, Kernel -> gendisk -> request queue순서이다.                                              
add_disk()가 호출된 후에는 /dev/mybrd 파일과 /sys/block/mybrd 디렉토리가 생긴다.                                        
*(add_disk()가 호출된 시점부터 디스크로 I/O가 발생하기 때문에 이전에 I/O를 처리할 준비가 되어있어야 한다.)               



--bio & bio_io_vec struct--                                                    
불연속하게 저장된 각각의 메모리 공간을 '세그먼트'라고 하며, 각 세그먼트는 bio_vec struct를 통해 관리된다.                  
bio struct는 bio_io_vec 필드를 통해 bio_vec struct를 배열로 저장하여 입출력 연산과 관련된 세그먼트들을 관리한다.
bio struct의 bi_sector 필드는 디스크 상에서 입출력 연산의 대상이 되는 시작주소를 가리킨다. bi_sector 필드는 bio가 초기 생성될 때에는 논리 블록 주소(LBA)로 설정되지만, request에 병합되기 전에 물리 블록 주소(PBA)로 변경된다. 논리블록주소는 디스크 파티션 내의 블록 인덱스를 의미하며, 물리 블록 주소는 실제 디스크 내에서의 블록 인덱스를 의미한다.                                                                                  
bio_vec struct는 bv_page 필드를 통해 세그먼트가 포함된 페이지를 가리키며, bv_offset과 bv_len필드를 통해 페이지 내의 세그먼트의 시작 위치와 크기를 나타낸다.(DMA가 page 단위로 실행되기 때문에 bio_vec이 page관련 정보를 담고있다.)
bio_for_each_segment()매크로를 이용해 bio_vec을 하나씩 꺼내와 처리한다. 이후 bio_endio()를 통해 할당된 bio객체를 해제한다.
*(더 자세한 내용은  https://www.kernel.org/doc/Documentation/block/biodoc.txt 를 참고)




드라이버 등록이 잘 되었으면 /dev/mybrd 파일과 /sys/block/mybrd 폴더가 생성 되었을 것이다.
/sys/block/mybrd 폴더에서 가장 중요한 파일은 'stat'파일로 이 장치에 얼마만큼의 I/O가 발생했는지를 기록하는 파일이다.
(자세한 정보는 https://www.kernel.org/doc/Documentation/block/stat.txt 를 참고)
또한 queue폴더는 앞서 정의한 request queue의 정보를 담고있다.         
(자세한 정보는 https://www.kernel.org/doc/Documentation/block/queue-sysfs.txt 를 참고)







