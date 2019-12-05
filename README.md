# ESTK보드와 ubinos운영체제를 활용하여 작성한 코드를 실행시킵니다.


1. 개요
함수 malloc을 사용하여 memory overflow를 유도하는 프로그램 

2. 프로그램 구조 설명
	2.1 함수에 대한 설명
` 	void * malloc(size_t size)
	  : Size를 인수로 받아 메모리의 heap영역에 size만큼의 void 포인터형의 공간을 할당한다.
	  <stdlib.h> 헤더 파일을 include 해야한다.
    
    
     void glcd_init(void)	
    : graphic LCD 초기화 함수. 
    
    
    void glcd_printf(char * format_p, ... )	
    : 문자를 graphic LCD에 출력하는 함수
    
    
    void glcd_clear(void)
	   : graphic LCD 화면을 지워준다.
     
     
	    int task_sleep(unsigned int tick)
	   : 실행에 Delay를 부여한다. 받은 인수의 시간만큼. 단위는 ms.


# 프로그램 소스파일
/*
  Copyright (C) 2009 Sung Ho Park
  Contact: ubinos.org@gmail.com

  This file is part of the exe_helloworld component of the Ubinos.

  GNU General Public License Usage
  This file may be used under the terms of the GNU
  General Public License version 3.0 as published by the Free Software
  Foundation and appearing in the file license_gpl3.txt included in the
  packaging of this file. Please review the following information to
  ensure the GNU General Public License version 3.0 requirements will be
  met: http://www.gnu.org/copyleft/gpl.html.

  GNU Lesser General Public License Usage
  Alternatively, this file may be used under the terms of the GNU Lesser
  General Public License version 2.1 as published by the Free Software
  Foundation and appearing in the file license_lgpl.txt included in the
  packaging of this file. Please review the following information to
  ensure the GNU Lesser General Public License version 2.1 requirements
  will be met: http://www.gnu.org/licenses/old-licenses/lgpl-2.1.html.

  Commercial Usage
  Alternatively, licensees holding valid commercial licenses may
  use this file in accordance with the commercial license agreement
  provided with the software or, alternatively, in accordance with the
  terms contained in a written agreement between you and rightful owner.
*/

/* -------------------------------------------------------------------------
	Include
 ------------------------------------------------------------------------- */
#include "../ubiconfig.h"

// standard c library include
#include <stdio.h>
#include <sam4e.h>
#include <stdlib.h>

// ubinos library include
#include "itf_ubinos/itf/bsp.h"
#include "itf_ubinos/itf/ubinos.h"
#include "itf_ubinos/itf/bsp_fpu.h"

// chipset driver include
#include "ioport.h"
#include "pio/pio.h"

// new estk driver include
#include "lib_new_estk_api/itf/new_estk_led.h"
#include "lib_new_estk_api/itf/new_estk_glcd.h"

// custom library header file include
//#include "../../lib_default/itf/lib_default.h"

// user header file include

/* -------------------------------------------------------------------------
	Global variables
 ------------------------------------------------------------------------- */

/* -------------------------------------------------------------------------
	Prototypes
 ------------------------------------------------------------------------- */
static void rootfunc(void * arg);


/* -------------------------------------------------------------------------
	Function Definitions
 ------------------------------------------------------------------------- */
int usrmain(int argc, char * argv[]) {
	int r;

	printf("\n\n\n\r");
	printf("================================================================================\n\r");
	printf("exe_ubinos_test (build time: %s %s)\n\r", __TIME__, __DATE__);
	printf("================================================================================\n\r");

	r = task_create(NULL, rootfunc, NULL, task_getmiddlepriority(), 256, "root");
	if (0 != r) {
		logme("fail at task_create\r\n");
	}

	ubik_comp_start();

	return 0;
}

static void rootfunc(void * arg) {

	glcd_init();
	//glcd_printf("hello world!\n\r");
	int *heap;
	int i=0;

		for(;;){

			heap = (int*)malloc(sizeof(double));	// int형 포인터 변수로 선언된 변수 heap에 malloc을 통해 생성된 배열의 주소 대입
			glcd_printf("iteration : %d \n", i);	// 현재 몇 번이나 malloc을 통해 공간을 할당하였는지 확인
			i++;
			task_sleep(10);							// 10ms만큼 delay를 부여

			if(heap == NULL){ 						// memory overflow 발생 시 해당 문자 출력 후 종료
				glcd_clear();
				glcd_printf("memory overflow \nat %d th iteration\n",i);
				break;
			}

		}
}






고찰: 

처음 생각했던 방법은 단순히 무한 반복되는 for문 안에 malloc를 사용하기만 하면 자동으로 lcd에 출력될 줄 알았다. 그러나 예상과는 다른 결과에 당황했다. 그리고 나서 의문이 들었던 점은 
[warning] heap_allocblock_bestfits: memory is not enough 라는 문구가 memory overflow가 발생하면 자동으로 lcd화면에 출력되는건지 아니면 console창에 출력되는건지 Error창이 따로 뜨는 건지 알 수가 없었다. 그렇게 고민을 하다 이클립스의 Search기능을 통해 heap_allocblock_bestfits를 검색하였더니 heap.c 파일,  malloc 함수 안에 
static void * heap_allocblock_bestfits(_heap_pt heap, unsigned int size, int direction) 라는 함수가 나왔다. 이 함수 안에  logmw("memory is not enough");  
이 부분이 포함되어 있었다. ("memory is not enough”)  이 부분이 우리가 확인해야할 메시지와 유사하여 logmw 함수를 보았다. 어떤 메시지를 기록하는 매크로 함수인 것을 알았다. 이 부분을 잘 활용하면 원하는 메시지를 출력할 수 있을 것 같았다. 그렇지만 결국 실패하였다. 그리고 나서 택한 방법이 가장 단순하게 무작정 실행하여 overflow발생 지점에서 메시지를 출력하자 하여 아래와 같은 코드를 작성하였다. 아쉬웠던 점은 [warning] 메시지를 출력하는데 있어서 가장 본질적인 부분을 놓친 듯한 느낌이 들었다.

