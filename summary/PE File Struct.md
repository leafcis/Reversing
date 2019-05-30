# PE(Portable Executable) File Structure



1. PE란?
2. PE File Format
3. PE Heder







## 1. PE란?

PE 파일은 Windows 운영체제에서 사용되는 실행 파일 형식이다.

UNIX에서 사용되는 COFF를 기반으로 MS에서 만들었는데, 애초에는 다른 운영체제에 이식성을 좋게 하려는 의도로 제작하였으나 실제로는 Windows 계열의 운영체제에서만 사용되고 있다.

PE 파일은 32비트 형태의 실행 파일을 의미하며 PE32라는 용어를 사용하기도 한다.

64비트 형태의 실행파일은 PE+ 또는 PE32+라고 부르며 PE파일의 확장 형태이다.



## 2. PE File Format

| 종류               | 주요 확장자        |
| ------------------ | ------------------ |
| 실행계열           | EXE, SCR           |
| 라이브러리 계열    | DLL, OCX, CPL, DRV |
| 드라이버 계열      | SYS, VXD           |
| 오브젝트 파일 계열 | OBJ                |

엄밀히 이야기하면 OBJ 파일을 제외한 모든 것은 실행 가능하다. DLL, SYS 파일 등은 셸에서는 직접 실행할 수 없으나 다른 방법에서는 실행 가능하다.

PE 헤더에는 실행에 필요한 DLL들과 필요한 stack/heap 메모리의 크기를 얼마로 할지 등등 수많은 정보가 구조체 형식으로 저장되어 있다.

<img src = "https://user-images.githubusercontent.com/36766295/58426952-1334d980-80d9-11e9-94df-61ff76fe1c17.png">



이 사진은 어떤 파일이 메모리에 적재(loading or mapping)될 때의 모습을 나타낸 것으로, 많은 내용을 함축하고 있다.

DOS header부터 Section header까지를 PE 헤더, 그 밑의 Section들을 합쳐서 PE 바디(Body)라고 한다.

파일에서는 offset, 메모리에서는 VA(Virtual Address, 절대주소)로 위치를 표현한다.

파일이 메모리에 로딩되면 모양이 달라지며(Section의 크기, 위치 등), 파일의 내용은 보통 코드(.text), 데이터(.data), 리소스(.rsrc) 섹션에 나뉘어서 저장된다.

> 개발도구와 빌드 옵션에 따라서 섹션의 이름, 크기, 개수, 저장 내용등은 달라지나, 중요한 것은 각 용도별로 여러 섹션이 나뉘어서 저장된다는 것이다.

섹션 헤더에 각 Section에 대한 파일/메모리에서의 크기, 위치, 속성 등이 정의되어 있다.

PE 헤더의 끝부분과 각 섹션의 끝에는 NULL padding이라고 불리우는 영역이 존재하는데, 컴퓨터에서 파일, 메모리, 네트워크 패킷 등을 처리할 때 효율을 높이기 위해 최소 기본 단위 개념을 사용하는 것처럼 PE 파일 에도 같은 개념이 적용된 것이다.

파일/메모리에서 섹션의 사직 위치는 각 파일/메모리의 최소 기본 단위의 배수에 해당하는 위치여야 하고, 빈 공간은 NULL로 채워버린다. 위 그림을 보면 각 세션의 시작 주소가 어떤 규칙에 의해 딱딱 끊어지는 걸 볼 수 있다.



### VA & RVA

VA랑 RVA 개념에 대해서 알고가자.

VA(Virtual Address)는 프로세스 가상 메모리의 절대 주소를 말하며, RVA(Relative Virtual Address)는 어느 기준 위치(Imagebase)에서부터의 상대주소를 말한다.

VA와 RVA의 관계는 `RVA + ImageBase = VA` 와 같은 식으로 나타낼 수 있다.

PE 헤더 내의 정보는 RVA 형태로 된 것이 많은데, 이는 PE파일이 프로세스 가상 메모리의 특정 위치에 로딩되는 순간 이미 그 위치에 다른 PE 파일이 로딩되어 있을 수 있기 때문이다. 이럴 때 재배치 과정을 통해서 비어 있는 다른 위치에 로딩되어야 하는데, 만약 PE 헤더 정보들이 VA로 되어 있다면 정상적인 엑세스가 이루어지지 않을 것이다. 그러므로 RVA를 통해서 재배치가 일어나도 기준위치에 대한 상대주소가 변하지 않기 때문에 아무런 문제없이 원하는 정보에 엑세스할 수 있게 되는 것이다.





## PE Header

PE 헤더는 많은 구조체로 이루어져 있다.



### DOS Header

MS에서 PE File Format을 만들 때 당시에 널리 사용되던 DOS 파일에 대한 하위 호환성을 고려해서 만들었다. 그 결과 PE 헤더의 제일 앞부분에는 기존 DOS EXE Header를 확장시킨 IMAGE_DOS_HEADER 구조체가 존재한다.



```c
typedef struct _IMAGE_DOS_HEADER {
    WORD e_magic;			// DOS signature : 4D5A ("MZ")
    WORD e_cblp;
    WORD e_cp;
    WORD e_crlc;
    WORD e_cparhdr;
    WORD e_minalloc;
    WORD e_maxalloc;
    WORD e_ss;
    WORD e_sp;
    WORD e_csum;
    WORD e_ip;
    WORD e_cs;
    WORD e_lfarlc;
    WORD e_ovno;
    WORD e_res[4];
    WORD e_oemid;
    WORD e_oeminfo;
    WORD e_res2[10];
    LONG e_lfanew;			// offset to NT header
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

IMAGE_DOS_HEADER 구조체의 크기는 40이다. 이 구조체에서 꼭 알아둬야 할 중요한 멤버는 e_magic과 e_lfanew이다.



> e_magic : DOS signature (4D5A => ASCII 값 "MZ")
>
> e_lfanew : NT header의 옵셋을 표시(파일에 따라 가변적인 값을 가짐)



모든 PE 파일은 시작 부분(e_magic)에 DOS signature ("MZ")가 존재하고, e_lfanew 값이 가리키는 위치에 NT Header 구조체가 존재해야 한다.

구조체의 이름은 IMAGE_NT_HEADER이다. 