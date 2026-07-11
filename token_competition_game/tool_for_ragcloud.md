내부 로컬 LLM 지식 구축



작성자 임대환

듣기

21

반응 추가
접속정보
hosts 설정 추가



172.30.1.160 rag.cloud.go.kr
 

페이지 접속

https://rag.cloud.go.kr/

image-20260605-061401.png
sign up : 회사 메일 계정으로 가입

팀장/파트장에게 가입된 아이디 전달

 

관리자

관리자 승인하여 팀 RAG 초대

 

Accept 클릭

image-20260612-054653.png
 

 

Model Provider 설정
image-20260612-041002.png
Model Provider 를 설정해야 Chat, Search 등을 이용 할 수 있음

 

GPUStack 연결 ( Embedding, Rerank )
API Key : gpustack_c0cafb6741aaec31_1dae62b2722d20b9774460c9935a14ca

Base URL : http://172.30.2.200:8080

Embedding Model

image-20260612-041222.png
Rerank Model

image-20260612-041328.png
 

 

OpenAI-API-Compatible 연결 ( LLM,Chat )
Base URL : http://172.30.2.200:8080/v1

API Key : gpustack_a6b9219d112c8b57_9679cad6a52db76c751fe3cc13548430

– LLM 모델은 Verify 를 여러번 눌러야 Valid 가 된다

 

LLM

image-20260612-041521.png


지식 구축 가이드



작성자 임대환

듣기

10

반응 추가
hosts 설정 추가



172.30.1.160 rag.cloud.go.kr
 

RAG 접속 ( https://rag.cloud.go.kr/ ) 

image-20260605-061913.png
Dataset 선택 후 파일 업로드

image-20260605-062023.png
 

업로드 파일은 Markdown 형태로 작성하여 업로드 바랍니다

 

업로드 지식 확인 ( Chunk Result )

image-20260605-062158.png