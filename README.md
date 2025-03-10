UiPath RPA 학습 시작 (개인 연습, 학습 공간)
----------------------------------

### Categories
          
- [RPA](#RPA)     
- [Abbyy Finereader]
- [Abbyy Flexicapture]

---------------------------------- 
  
### Main   
- BlockUserInput 사용 - 디버그 또는 실행중 마우스 움직임 제어
- StateMachine - Init -> Get Transaction Data -> Process Transaction -> End Process

![image](https://user-images.githubusercontent.com/53161059/133537592-21c712ee-05fe-4f9a-ae74-503b6c121663.png)
 
----------------------------------- 

 
#### Init
- Config 파일 세팅, Kill 프로세스, 기본 프로그램 실행 세팅
#### Get Transaction Data
- 데이터 read 후 하나의 Data 만 발췌
#### Process Transaction
- 발췌 된 Data 1개 실행 프로세스
#### End Process
- 프로세스 종료

--------------------------------------

##### Init
Config 파일 세팅, Kill 프로세스, 기본 프로그램 실행 세팅
- Start ->
- Init Success -> Get Transaction Data
- Init Error -> End Process
- Error & Retry <- Process Transaction
##### Get Transaction Data
데이터 read 후 하나의 Data 만 발췌
- Init Success <- Init
- Get Success -> Process Transaction
- Success <- Process Transaction
- Rule Error <- Process Transaction
- No Data -> End Process
##### Process Transaction
발췌 된 Data 1개 실행 프로세스
- Get Success <- Get Transaction Data
- Success -> Get Transaction Data
- Rule Error -> Get Transaction Data
- Error & Retry -> Init
- Error & NoRetry -> End Process
##### End Process
프로세스 종료
- Init Error
- No Data
- Error & NoRetry

---------------------------------------------

## Init 
Config 파일 세팅, Kill 프로세스, 기본 프로그램 실행 세팅
- Start ->
  - Try 안에 
    - if 조건 Config is Nothing  부합하면
    - Invoke workflow (InitAllSettings) - 엑셀 config 파일 dictionary 설정, for each <string> 이용해 엑셀 데이터 추출
    - AppendLine 프로젝트 시작 로그 작성
    - Invoke workflow (KillAllProcess) - 실행 및 디버그 전 불필요한 프로세스 닫기
	
    - If 조건 TransactionNumber=1 And RetryNumber=0 부합하지 않으면
    - 재시작 로그메세지  
	        
    - 시간 로그메세지
    - Invoke workflow (KillAllProcess) - 실행 및 디버그 전 불필요한 프로세스 닫기
	
      - Catch 안에
        - SystemError exception
	
      - Init Success -> Get Transaction Data
        - 조건 SystemError is Nothing
        - 시간 로그 작성

      - Init Error -> End Process
        - 조건 SystemError isNot Nothing
        - 에러 로그 작성 
	 
      - Error & Retry <- Process Transaction
        - 조건 에러 및 Retry 
        - 에러 로그 작성
	

## Get Transaction Data
데이터 read 후 하나의 Data 만 발췌
- Try 안에 시작 로그메세지 (DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")+"----------Data "+TransactionNumber.ToString+" : GET TransactionData Start")
  - if 조건 SystemError is Nothing
  - 부합하면 Invoke workflow (GetTransactionData)
    - GetTransactionData = 최초 엑셀작업, 데이터테이블 핸들링, 엑셀 데이터 한줄씩 읽어오기
    - 부합하지 않으면 Throw
- Catch 안에 Exception
  - 로그 메시지	
  - TransactionData = Nothing

    - Init Success <- Init
    - Get Success -> Process Transaction
      - 조건 TransactionData IsNot Nothing
      - try 로그메세지
      - catch SystemError = exception, 로그메세지

    - Success <- Process Transaction
    - Rule Error <- Process Transaction
    - No Data -> End Process
      - 조건 TransactionData Is Nothing
      - try 로그 메시지	
      - catch SystemError = exception
      - 로그 메세지
	

## Process Transaction
발췌 된 Data 1개 실행 프로세스
- Try 안에 BusinessRuleException = Nothing, 로그메세지
  - if 조건 SystemError Is Nothing
  - 부합하면 Invoke workflow(Process)
  - Process = 작업하는 액티비티들	
  - 부합하지 않으면 Throw
- Catches 안에 
  - BusinessRuleException = exception, Exception 에 SystemError = exception
- Finally 안에 
  - Try Invoke workflow(SetTransactionStatus)
  - Catches exception 로그메세지

  - Get Success <- Get Transaction Data
  - Success -> Get Transaction Data
	    조건 SystemError Is Nothing 
	      
  - Rule Error -> Get Transaction Data
	    조건 SystemError Is Nothing And BusinessRuleException is Nothing
	      catches  exception SystemError = exception, 로그 메세지
	
  - Error & Retry -> Init
	    조건 SystemError IsNot Nothing And Config("FlagRetry").ToString.ToLower.Equals("true")
	    에러 메시지 
	
  - Error & NoRetry -> End Process
	    조건 SystemError IsNot Nothing And Config("FlagRetry").ToString.ToLower.Equals("false")
	    에러 메세지
	

## End Process
프로세스 종료
- Try 안에 if 조건 SystemError IsNot Nothing
- 부합하면 Throw
- 부합하지 않으면 성공적으로 처리하는(Invoke workflow ) 후(KillProcess 추가) 로그 메세지

  - Init Error <- Init
  - No Data <- Get Transaction Data 
  - Error & NoRetry <- Process Transaction

---------------------------
## a b
		     

