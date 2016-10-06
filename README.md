---
title: "보험 사기 예측"
author: "yunjihwan"
date: "2016년 9월 1일"
output: html_document
---



## 빅콘테스트
  * 빅데이터 문제를 기계학습에 기반하여 직접 체험 및 분석하는 기회를 제공함으로써, 현안 문제를 해결할 수 있는 빅데이터 전문가를 발굴하고 취업 연계 지원 
  * 기간 8.1 ~ 9.30
  * http://contest.kbig.kr/
  

###  챌린지리그
  
  * 빅데이터 분석을 통한 보험사기 예측 알고리즘 개발
  * 최근 약 10여년 간의 보험사기자와 일반고객으로 구분하여 구성된 데이터공 제공
  * 고객정보, 계약정보, 지급정보, 설계사 정보 등 포함
  
  * 결과 및 평가
    * Test-set 내에서의 보험사기자 및 일반고객 구분 결과표
    * F-measure 값(Precision/Recall의 조화평균)을 비교하여 높은 참가자 순으로 2차 평가 대상자 선정
    * Precision: 참가자가 보험사기자로 판별한 사람 중 실제 보험사기자의 비율
    * Recall: 실제보험사기자 전체 중 참가자가 보험사기자로 판별한 사람의 비율 
    
    
###  데이터 소개
    
  * BGCON_CUST_DATA
    * 보험 계약자 관련 데이터
    * CUST_ID(고객 번호) 기준으로 개인별 1개씩 필드를 가지고 있음
    * CUST_ID기준으로 BGCON_CLAIM_DATA, BGCON_CNTT_DATA, BGCON_FMLY_DATA 데이터와 연결되어 있음
    
  * BGCON_CLAIM_DATA
    * 보험 신청 관련 데이터음
    * CUST_ID(고객 번호) 기준으로 여러건의 보험 청구 건수를 가지고 있으
    
  * BGCON_CNTT_DATA
    * 보험 계약 관련 데이터
    * CUST_ID(고객 번호) 기준으로 여러건의 보험계약사항을 가지고 있음
    * CLLT_FP_PRNO(FP사번)을 기준으로 BGCON_FPINFO_DATA와 연결되어 있음

  * BGCON_FMLY_DATA
    * 보험자간 관계 데이터음
    * CUST_ID(고객 번호) - SUB_CUST_ID(고객 번호) 번호를 기준으로 고객간의 관계에 대해 표시
    
  * BGCON_FPINFO_DATA
    * 보험설계사(FP) 관련 데이터
    * CLLT_FP_PRNO(FP사번) 기준으로 BGCON_CNTT_DATA와 연결되어 있음.
    * 한 FP는 여러 보험 계약 건수를 가지고 있음

#### 목적은 BGCON_CUST_DATA 데이터의  보험사기여부(SIU_CUST_YN)필드의 보험사기(Y) 보험사기아님(N)을 분류하는 문제


### 탐색적 데이터 분석

#### 결측치 확인

  * 실효년월(LAPS_YM) 이 대부분 비어 있음
  * 소멸년월(EXTN_YM) 많이 비어 있음
  * FP와 고객간의거리(DISTANCE)는 일부 비어 있음
  * MAXCRDT, MINCRDT는 중요한 factor로 보이지만 많은 데이터가 비어 있음

#### SIU_CUST_YN 보험사기자를 기준으로 데이터를 확인
#### BGCON_CUST_DATA

  * FP경력(FP_CAREER)이 5% 더 높음 하지만, 모수가 많지 않음

  * 신용등급_최소(MINCRDT)의 경우 7등급 이하의 하위가 더 높음
  * 신용등급_최대(MAXCRDT)의 경우 7등급 이하의 하위가 더 높음

  * 결혼여부(WEDD_YN) 결혼한 경우가 4% 더 높음

  * FMLY_RELN_CODE 형제 자매의 관계에서 사기로 분류된 경우가 많음, 가족 아이디로 엮인 부분이 더 많음


#### BGCON_CLAIM_DATA

  * CLAIM 데이터의 경우, 보험처리건수로 여러개일 수 있기 때문에 중복 집계될 가능성이 높음
  * 좀더 정확히 보기 위해, 보험처리건수별 아이디 한개로 제한해서 테스트 해야함

  * 직업그룹코드1(ACCI_OCCP_GRP1)
    * 1.주부인 경우 6% 더 높음
    * 자영업 경우 6% 더 높음
    * 서비스직인 경우 3% 더 높음
    * 전문직인 경우 3% 더 낮음
    * 제조업인 경우 3% 더 낮음
  
  * 직업그룹코드2(ACCI_OCCP_GRP2)
    * 3차산업 종사자 경우 5% 더 높음
    * 1차산업종사자 1%, 2차산업종사자 3% 더 낮음
    * 자영업 경우 5% 더 높음
    * 주부 경우 6% 더 높음
    * 법무직 종사자의 경우 사기건수 미존재
    * 고소득 전문직의 경우 사기건수 미존재
  
  * 청구사유 코드(DMND_RESN_CODE)
    * 02(입원)이 유의미하게 높음 23%
    * 대신 수술(05)는 절반정도 낮음
  
  * 실소처리여부(PMMI_DLNG_YN)은 현격하게 낮음 15%
  
  * 금감원 유의병원 대상(HEED_HOSP_YN)유의 병원인 경우가 2%더 높음
  
  * 병원종별구분(HOSP_SPEC_DVSN)에서
    * 한방병원(80) 8%, 의료기관이외(95) 7%로 유의미 하게 높음
    * 종합병원(10) 20%로 유의미 하게 낮음
  
  * 유효입원/통원일수(VLID_HOSP_OTDA) 최소, 평균, 최대 모두 높게 나옴

  * 뉴스기사 : 지난해 보험사기 적발 6549억원…사상 최대
    * 위 뉴스기사와 비슷한 결과를 보이는 factor들이 많이 존재
    * 주부, 자영업의 직업군 영향이 높

#### BGCON_CNTT_DATA

  * FP경력(FP_CAREER)이 5% 더 높음 하지만, 모수가 많지 않음

  * 신용등급_최소(MINCRDT)의 경우 7등급 이하의 하위가 더 높음
  * 신용등급_최대(MAXCRDT)의 경우 7등급 이하의 하위가 더 높음

  * 결혼여부(WEDD_YN) 결혼한 경우가 4% 더 높음

  * FMLY_RELN_CODE 형제 자매의 관계에서 사기로 분류된 경우가 많음, 가족 아이디로 엮인 부분이 더 많음


### 상관분석

  * 의미 있는 상관관계 파악



```{r}
#Set path
setwd("C:\\xxxxx\\xxxxx\\xxxx")

#######################################################################
########################### PRE PROCESSING ############################
#######################################################################
#라이브러리 불러오기
#install.packages("corrplot")
suppressPackageStartupMessages({
  library(caret)
  library(corrplot)			# plot correlations
  library(doParallel)		# parallel processing
  library(dplyr)        # Used by caret
  library(gbm)				  # GBM Models
  library(pROC)				  # plot the ROC curve
  library(xgboost)      # Extreme Gradient Boosting
  library(missForest)   # Missing Data 채우는 패키지
  library(caretEnsemble)
  library(DMwR)
  library(randomForest) # for feature importance
})

###################### Set up for parallel procerssing
set.seed(13784)
registerDoParallel(4,cores=4)
getDoParWorkers()


# CSV Raw Data 불러오기
BGCON_CLAIM_DATA <- read.csv("data/BGCON_CLAIM_DATA.csv")
BGCON_CNTT_DATA <- read.csv("data/BGCON_CNTT_DATA.csv")
BGCON_CUST_DATA <- read.csv("data/BGCON_CUST_DATA.csv") # 기준 데이터
BGCON_FMLY_DATA <- read.csv("data/BGCON_FMLY_DATA.csv")
BGCON_FPINFO_DATA <- read.csv("data/BGCON_FPINFO_DATA.csv")


############################################ Train Test, 정확히 분리하기
# 제출용 Test 데이터를 분리
BGCON_CUST_DATA.TRAIN <- BGCON_CUST_DATA[which(BGCON_CUST_DATA$DIVIDED_SET == 1), ]
BGCON_CUST_DATA.TEST <- BGCON_CUST_DATA[which(BGCON_CUST_DATA$DIVIDED_SET == 2), ]

############################# TRAIN ###################################
# CLAIM 데이터에서 CUST ID에 해당하는 ROW들만 추출해서 새로운 DF로 저장함
#cust_id <- data.frame(CUST_ID=train.batch.down$CUST_ID) 
#BGCON_CLAIM_DATA.TRAIN <- merge(x=BGCON_CLAIM_DATA, y=cust_id, by="CUST_ID", All.x=TRUE)
BGCON_CLAIM_DATA.TRAIN <- BGCON_CLAIM_DATA[BGCON_CLAIM_DATA$CUST_ID %in% BGCON_CUST_DATA.TRAIN$CUST_ID, ] #위 2줄과 동일한 효과
#for debugging
#head(unique(sort(train.batch.down$CUST_ID)), n = 10)
#head(unique(sort(BGCON_CLAIM_DATA.TRAIN$CUST_ID)), n = 10)

# BGCON_FMLY_DATA
BGCON_FMLY_DATA.TRAIN <- BGCON_FMLY_DATA[BGCON_FMLY_DATA$CUST_ID %in% BGCON_CUST_DATA.TRAIN$CUST_ID, ] 
#for debugging
#head(unique(sort(train.batch.down$CUST_ID)), n = 100)
#head(unique(sort(BGCON_FMLY_DATA.TRAIN$CUST_ID)), n = 100)

# BGCON_CNTT_DATA
BGCON_CNTT_DATA.TRAIN <- BGCON_CNTT_DATA[BGCON_CNTT_DATA$CUST_ID %in% BGCON_CUST_DATA.TRAIN$CUST_ID, ] 
#for debugging
#head(unique(sort(train.batch.down$CUST_ID)), n = 10)
#head(unique(sort(BGCON_CNTT_DATA.TRAIN$CUST_ID)), n = 10)

# BGCON_FPINFO_DATA
BGCON_FPINFO_DATA.TRAIN <- BGCON_FPINFO_DATA[BGCON_FPINFO_DATA$CLLT_FP_PRNO %in% BGCON_CNTT_DATA.TRAIN$CLLT_FP_PRNO, ]
#for debugging
#head(unique(sort(BGCON_CNTT_DATA.TRAIN$CLLT_FP_PRNO)), n = 10)
#head(unique(sort(BGCON_FPINFO_DATA.TRAIN$CLLT_FP_PRNO)), n = 10)

############################# TEST ###################################
# CLAIM 데이터에서 CUST ID에 해당하는 ROW들만 추출해서 새로운 DF로 저장함
BGCON_CLAIM_DATA.TEST <- BGCON_CLAIM_DATA[BGCON_CLAIM_DATA$CUST_ID %in% BGCON_CUST_DATA.TEST$CUST_ID, ]
BGCON_FMLY_DATA.TEST <- BGCON_FMLY_DATA[BGCON_FMLY_DATA$CUST_ID %in% BGCON_CUST_DATA.TEST$CUST_ID, ] 
BGCON_CNTT_DATA.TEST <- BGCON_CNTT_DATA[BGCON_CNTT_DATA$CUST_ID %in% BGCON_CUST_DATA.TEST$CUST_ID, ] 
BGCON_FPINFO_DATA.TEST <- BGCON_FPINFO_DATA[BGCON_FPINFO_DATA$CLLT_FP_PRNO %in% BGCON_CNTT_DATA.TEST$CLLT_FP_PRNO, ]
#head(unique(sort(BGCON_CNTT_DATA.TEST$CLLT_FP_PRNO)), n = 10)
#head(unique(sort(BGCON_FPINFO_DATA.TEST$CLLT_FP_PRNO)), n = 10)


############################# TRAIN ###################################
####DATE 변수들에서 YEAR를 추출함
#BGCON_CLAIM_DATA
BGCON_CLAIM_DATA.TRAIN$RECP_DATE_YEAR <- as.numeric(format(as.Date(as.character(BGCON_CLAIM_DATA.TRAIN$RECP_DATE), format = "%Y%m%d"), "%Y"))
BGCON_CLAIM_DATA.TRAIN$ORIG_RESN_DATE_YEAR <- as.numeric(format(as.Date(as.character(BGCON_CLAIM_DATA.TRAIN$ORIG_RESN_DATE), format = "%Y%m%d"), "%Y"))
BGCON_CLAIM_DATA.TRAIN$RESN_DATE_YEAR <- as.numeric(format(as.Date(as.character(BGCON_CLAIM_DATA.TRAIN$RESN_DATE), format = "%Y%m%d"), "%Y"))
BGCON_CLAIM_DATA.TRAIN$PAYM_DATE_YEAR <- as.numeric(format(as.Date(as.character(BGCON_CLAIM_DATA.TRAIN$PAYM_DATE), format = "%Y%m%d"), "%Y"))

#BGCON_CLAIM_DATA
BGCON_CUST_DATA.TRAIN$CUST_RGST_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CUST_DATA.TRAIN$CUST_RGST), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CUST_DATA.TRAIN$MAX_PAYM_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CUST_DATA.TRAIN$MAX_PAYM_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))

#BGCON_CNTT_DATA
BGCON_CNTT_DATA.TRAIN$CNTT_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CNTT_DATA.TRAIN$CNTT_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CNTT_DATA.TRAIN$EXPR_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CNTT_DATA.TRAIN$EXPR_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CNTT_DATA.TRAIN$EXTN_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CNTT_DATA.TRAIN$EXTN_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CNTT_DATA.TRAIN$LAPS_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CNTT_DATA.TRAIN$LAPS_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))

#BGCON_FPINFO_DATA
BGCON_FPINFO_DATA.TRAIN$ETRS_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_FPINFO_DATA.TRAIN$ETRS_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_FPINFO_DATA.TRAIN$FIRE_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_FPINFO_DATA.TRAIN$FIRE_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))

##BGCON_CLAIM_DATA 데이터에서 병명 데이터 앞글자를 분리 
BGCON_CLAIM_DATA.TRAIN$CAUS_CODE_FIRST <- substring(BGCON_CLAIM_DATA.TRAIN$CAUS_CODE, 1, 1)
BGCON_CLAIM_DATA.TRAIN$RESL_CD1_FIRST <- substring(BGCON_CLAIM_DATA.TRAIN$RESL_CD1, 1, 1)


############################# TEST ###################################
####DATE 변수들에서 YEAR를 추출함
BGCON_CLAIM_DATA.TEST$RECP_DATE_YEAR <- as.numeric(format(as.Date(as.character(BGCON_CLAIM_DATA.TEST$RECP_DATE), format = "%Y%m%d"), "%Y"))
BGCON_CLAIM_DATA.TEST$ORIG_RESN_DATE_YEAR <- as.numeric(format(as.Date(as.character(BGCON_CLAIM_DATA.TEST$ORIG_RESN_DATE), format = "%Y%m%d"), "%Y"))
BGCON_CLAIM_DATA.TEST$RESN_DATE_YEAR <- as.numeric(format(as.Date(as.character(BGCON_CLAIM_DATA.TEST$RESN_DATE), format = "%Y%m%d"), "%Y"))
BGCON_CLAIM_DATA.TEST$PAYM_DATE_YEAR <- as.numeric(format(as.Date(as.character(BGCON_CLAIM_DATA.TEST$PAYM_DATE), format = "%Y%m%d"), "%Y"))
BGCON_CUST_DATA.TEST$CUST_RGST_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CUST_DATA.TEST$CUST_RGST), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CUST_DATA.TEST$MAX_PAYM_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CUST_DATA.TEST$MAX_PAYM_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CNTT_DATA.TEST$CNTT_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CNTT_DATA.TEST$CNTT_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CNTT_DATA.TEST$EXPR_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CNTT_DATA.TEST$EXPR_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CNTT_DATA.TEST$EXTN_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CNTT_DATA.TEST$EXTN_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_CNTT_DATA.TEST$LAPS_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_CNTT_DATA.TEST$LAPS_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_FPINFO_DATA.TEST$ETRS_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_FPINFO_DATA.TEST$ETRS_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
BGCON_FPINFO_DATA.TEST$FIRE_YM_YEAR <- as.numeric(format(as.Date(paste(as.character(BGCON_FPINFO_DATA.TEST$FIRE_YM), c("01"), sep=""), format = "%Y%m%d"), "%Y"))
##BGCON_CLAIM_DATA 데이터에서 병명 데이터 앞글자를 분리 
BGCON_CLAIM_DATA.TEST$CAUS_CODE_FIRST <- substring(BGCON_CLAIM_DATA.TEST$CAUS_CODE, 1, 1)
BGCON_CLAIM_DATA.TEST$RESL_CD1_FIRST <- substring(BGCON_CLAIM_DATA.TEST$RESL_CD1, 1, 1)











####다른 필드의 값을 Sum 하여 df1 필드에 채워줌
#df_from : 가져오려는 필드가 있는 df
#df_to : df_from에서 가져온 필드를 저장하려는 df
#t_col_name : df_from에서 가져오려 column 명
#c_col_name : 만들려는 column 이름
#by_col_name : merge 하기위해서 기준이 되는 column 명
#FUN : 적용할 function  ex) sum
#t_na : NA값 이 생길경우 넣을 값  ex)숫자의 경우 0으로 채움
merge_after_aggregate <- function(df_from, df_to, by_col_name, t_col_name, c_col_name, FUN, t_na=NULL){
  agg_df_temp <- aggregate(df_from[, t_col_name], by=list(df_from[, by_col_name]), FUN=FUN)
  colnames(agg_df_temp) <- c(by_col_name, c_col_name)
  ret_df <- merge(x = df_to, y = agg_df_temp, by = by_col_name, all=TRUE)
  #NA 값 채우기
  if(!is.null(t_na)){
    ret_df[is.na(ret_df[, c_col_name]), c_col_name] <- t_na
  }
  return (ret_df)
}

############################# TRAIN ###################################
## 각 아이디별 총 CLAIM(보험처리건수)를 기록하여 BGCON_CUST_DATA에 merge 함 
is.not.na <- function(x){ sum(!is.na(x)) }
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "POLY_NO", "CLAIM_TOTAL", is.not.na, 0)

## BGCON_CLAIM_DATA의 numeric 값의 데이터중 TOTAL(SUM)이 가능한 값들을 SUM하여 BGCON_CUST_DATA에 merge 함
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "VLID_HOSP_OTDA", "VLID_HOSP_OTDA_TOTAL", sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "VLID_HOSP_OTDA", "VLID_HOSP_OTDA_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "VLID_HOSP_OTDA", "VLID_HOSP_OTDA_MEDIAN", median)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "HOUSE_HOSP_DIST", "HOUSE_HOSP_DIST_TOTAL", sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "HOUSE_HOSP_DIST", "HOUSE_HOSP_DIST_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "HOUSE_HOSP_DIST", "HOUSE_HOSP_DIST_MEDIAN", median)
#정규분포로 정규화 하기위해 log + 1을 해줌
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DMND_AMT", "DMND_AMT_TOTAL", function(x){log(sum(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DMND_AMT", "DMND_AMT_MEAN",  function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DMND_AMT", "DMND_AMT_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "PAYM_AMT", "PAYM_AMT_TOTAL", function(x){log(sum(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "PAYM_AMT", "PAYM_AMT_MEAN", function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "PAYM_AMT", "PAYM_AMT_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "SELF_CHAM", "SELF_CHAM_TOTAL", sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "SELF_CHAM", "SELF_CHAM_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "SELF_CHAM", "SELF_CHAM_MEDIAN", median)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "NON_PAY", "NON_PAY_TOTAL", sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "NON_PAY", "NON_PAY_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "NON_PAY", "NON_PAY_MEDIAN", median)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "TAMT_SFCA", "TAMT_SFCA_TOTAL", sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "TAMT_SFCA", "TAMT_SFCA_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "TAMT_SFCA", "TAMT_SFCA_MEDIAN", median)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "PATT_CHRG_TOTA", "PATT_CHRG_TOTA_TOTAL",sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "PATT_CHRG_TOTA", "PATT_CHRG_TOTA_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "PATT_CHRG_TOTA", "PATT_CHRG_TOTA_MEDIAN", median)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DSCT_AMT", "DSCT_AMT_TOTAL", sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DSCT_AMT", "DSCT_AMT_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DSCT_AMT", "DSCT_AMT_MEDIAN", median)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "COUNT_TRMT_ITEM", "COUNT_TRMT_ITEM_TOTAL",sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "COUNT_TRMT_ITEM", "COUNT_TRMT_ITEM_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "COUNT_TRMT_ITEM", "COUNT_TRMT_ITEM_MEDIAN", median)


## 각 아이디별 총 CNTT(보험가입건수)를 기록하여 BGCON_CUST_DATA에 merge 함 
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "POLY_NO", "CNTT_TOTAL", is.not.na, 0)

## BGCON_CNTT_DATA의 numeric 값의 데이터중 TOTAL(SUM)이 가능한 값들을 SUM하여 BGCON_CUST_DATA에 merge 함
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "REAL_PAYM_TERM", "REAL_PAYM_TERM_TOTAL",sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "REAL_PAYM_TERM", "REAL_PAYM_TERM_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "REAL_PAYM_TERM", "REAL_PAYM_TERM_MEDIAN", median)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "MAIN_INSR_AMT", "MAIN_INSR_AMT_TOTAL", function(x){log(sum(as.numeric(x))+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "MAIN_INSR_AMT", "MAIN_INSR_AMT_MEAN", function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "MAIN_INSR_AMT", "MAIN_INSR_AMT_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "SUM_ORIG_PREM", "SUM_ORIG_PREM_TOTAL", function(x){log(sum(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "SUM_ORIG_PREM", "SUM_ORIG_PREM_MEAN", function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "SUM_ORIG_PREM", "SUM_ORIG_PREM_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "MNTH_INCM_AMT", "MNTH_INCM_AMT_TOTAL", function(x){log(sum(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "MNTH_INCM_AMT", "MNTH_INCM_AMT_MEAN", function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "MNTH_INCM_AMT", "MNTH_INCM_AMT_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DISTANCE", "DISTANCE_TOTAL", sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DISTANCE", "DISTANCE_MEAN", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "DISTANCE", "DISTANCE_MEDIAN", median)

## 부동산금액 정규화
BGCON_CUST_DATA.TRAIN <- transform(BGCON_CUST_DATA.TRAIN, TOTALPREM_LOG = log(TOTALPREM + 1))
BGCON_CUST_DATA.TRAIN <- transform(BGCON_CUST_DATA.TRAIN, MAX_PRM_LOG = log(MAX_PRM + 1))

## 정규 분포가 적절한지 히스토그램으로 확인해서 적용함.
#summary(BGCON_CUST_DATA$TOTALPREM_LOG)
#hist(BGCON_CUST_DATA$MAX_PRM_LOG, breaks=50, freq=TRUE) # INCOME 히스토그램



############################# TEST ###################################
## 각 아이디별 총 CLAIM(보험처리건수)를 기록하여 BGCON_CUST_DATA에 merge 함 
is.not.na <- function(x){ sum(!is.na(x)) }
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "POLY_NO", "CLAIM_TOTAL", is.not.na, 0)

## BGCON_CLAIM_DATA의 numeric 값의 데이터중 TOTAL(SUM)이 가능한 값들을 SUM하여 BGCON_CUST_DATA에 merge 함
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "VLID_HOSP_OTDA", "VLID_HOSP_OTDA_TOTAL", sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "VLID_HOSP_OTDA", "VLID_HOSP_OTDA_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "VLID_HOSP_OTDA", "VLID_HOSP_OTDA_MEDIAN", median)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "HOUSE_HOSP_DIST", "HOUSE_HOSP_DIST_TOTAL", sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "HOUSE_HOSP_DIST", "HOUSE_HOSP_DIST_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "HOUSE_HOSP_DIST", "HOUSE_HOSP_DIST_MEDIAN", median)
#정규분포로 정규화 하기위해 log + 1을 해줌
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DMND_AMT", "DMND_AMT_TOTAL", function(x){log(sum(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DMND_AMT", "DMND_AMT_MEAN",  function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DMND_AMT", "DMND_AMT_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "PAYM_AMT", "PAYM_AMT_TOTAL", function(x){log(sum(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "PAYM_AMT", "PAYM_AMT_MEAN", function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "PAYM_AMT", "PAYM_AMT_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "SELF_CHAM", "SELF_CHAM_TOTAL", sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "SELF_CHAM", "SELF_CHAM_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "SELF_CHAM", "SELF_CHAM_MEDIAN", median)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "NON_PAY", "NON_PAY_TOTAL", sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "NON_PAY", "NON_PAY_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "NON_PAY", "NON_PAY_MEDIAN", median)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "TAMT_SFCA", "TAMT_SFCA_TOTAL", sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "TAMT_SFCA", "TAMT_SFCA_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "TAMT_SFCA", "TAMT_SFCA_MEDIAN", median)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "PATT_CHRG_TOTA", "PATT_CHRG_TOTA_TOTAL",sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "PATT_CHRG_TOTA", "PATT_CHRG_TOTA_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "PATT_CHRG_TOTA", "PATT_CHRG_TOTA_MEDIAN", median)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DSCT_AMT", "DSCT_AMT_TOTAL", sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DSCT_AMT", "DSCT_AMT_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DSCT_AMT", "DSCT_AMT_MEDIAN", median)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "COUNT_TRMT_ITEM", "COUNT_TRMT_ITEM_TOTAL",sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "COUNT_TRMT_ITEM", "COUNT_TRMT_ITEM_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "COUNT_TRMT_ITEM", "COUNT_TRMT_ITEM_MEDIAN", median)


## 각 아이디별 총 CNTT(보험가입건수)를 기록하여 BGCON_CUST_DATA에 merge 함 
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "POLY_NO", "CNTT_TOTAL", is.not.na, 0)

## BGCON_CNTT_DATA의 numeric 값의 데이터중 TOTAL(SUM)이 가능한 값들을 SUM하여 BGCON_CUST_DATA에 merge 함
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "REAL_PAYM_TERM", "REAL_PAYM_TERM_TOTAL",sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "REAL_PAYM_TERM", "REAL_PAYM_TERM_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "REAL_PAYM_TERM", "REAL_PAYM_TERM_MEDIAN", median)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "MAIN_INSR_AMT", "MAIN_INSR_AMT_TOTAL", function(x){log(sum(as.numeric(x))+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "MAIN_INSR_AMT", "MAIN_INSR_AMT_MEAN", function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "MAIN_INSR_AMT", "MAIN_INSR_AMT_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "SUM_ORIG_PREM", "SUM_ORIG_PREM_TOTAL", function(x){log(sum(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "SUM_ORIG_PREM", "SUM_ORIG_PREM_MEAN", function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "SUM_ORIG_PREM", "SUM_ORIG_PREM_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "MNTH_INCM_AMT", "MNTH_INCM_AMT_TOTAL", function(x){log(sum(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "MNTH_INCM_AMT", "MNTH_INCM_AMT_MEAN", function(x){log(mean(x)+1)})
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "MNTH_INCM_AMT", "MNTH_INCM_AMT_MEDIAN", function(x){log(median(x)+1)})

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DISTANCE", "DISTANCE_TOTAL", sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DISTANCE", "DISTANCE_MEAN", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "DISTANCE", "DISTANCE_MEDIAN", median)

## 부동산금액 정규화
BGCON_CUST_DATA.TEST <- transform(BGCON_CUST_DATA.TEST, TOTALPREM_LOG = log(TOTALPREM + 1))
BGCON_CUST_DATA.TEST <- transform(BGCON_CUST_DATA.TEST, MAX_PRM_LOG = log(MAX_PRM + 1))


#df_from : 가져오려는 필드가 있는 df
#df_to : df_from에서 가져온 필드를 저장하려는 df
#t_col_name : df_from에서 가져오려 column 명
#by_col_name : merge 하기위해서 기준이 되는 column 명
#t_na : NA값 이 생길경우 넣을 값  ex)숫자의 경우 0으로 채움
add_count_of_value_by_colname <- function(df_from, df_from2, df_to, t_col_name, by_col_name, t_na=NULL){
  
  df_from[, t_col_name] <- factor(df_from[, t_col_name])
  df_from2[, t_col_name] <- factor(df_from2[, t_col_name])
  
  #t_col_arr <- levels(df_from[, t_col_name])
  t_col_arr <- unique(c(levels(df_from[, t_col_name]), levels(df_from2[, t_col_name])))
  levels(df_from[, t_col_name]) <- t_col_arr
  cat("levels(df_from[, t_col_name])", levels(df_from[, t_col_name]), "\n")
  cat("levels(df_from2[, t_col_name])", levels(df_from2[, t_col_name]), "\n")
  cat(t_col_arr, "\n")
  
  for (t_val in t_col_arr){
    
    col_temp <- paste(t_col_name, "_", t_val, sep="")
    
    if(nrow(df_from[which(df_from[, t_col_name]== t_val), ]) > 0){
      
      agg_temp <- aggregate(df_from[which(df_from[, t_col_name]== t_val), t_col_name], by=list(df_from[which(df_from[, t_col_name] == t_val), by_col_name]), FUN=function(x){ sum(!is.na(x))
      })
      
      colnames(agg_temp) <- c(by_col_name, col_temp)
      
      df_to <- merge(x = df_to, y = agg_temp, by = by_col_name, all=TRUE)
      
      #NA 
      if(!is.null(t_na)){
        df_to[is.na(df_to[, col_temp]), col_temp] <- t_na
      }
    }else{
      
      df_to[, col_temp] <- NA
      
      #NA 
      if(!is.null(t_na)){
        df_to[is.na(df_to[, col_temp]), col_temp] <- t_na
      }
    }
  }
  return (df_to)
}


############################# TRAIN ###################################
## BGCON_CUST_DATA의  YEAR로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CUST_DATA.TRAIN, BGCON_CUST_DATA.TEST, BGCON_CUST_DATA.TRAIN, "CUST_RGST_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CUST_DATA.TRAIN, BGCON_CUST_DATA.TEST, BGCON_CUST_DATA.TRAIN, "MAX_PAYM_YM_YEAR", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 YEAR로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "RECP_DATE_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "ORIG_RESN_DATE_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "RESN_DATE_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "PAYM_DATE_YEAR", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 YEAR로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "CNTT_YM_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "EXPR_YM_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "EXTN_YM_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "LAPS_YM_YEAR", "CUST_ID", 0)

## BGCON_FPINFO_DATA에서 YEAR로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CNTT_DATA에 merge함
BGCON_CNTT_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TRAIN, BGCON_FPINFO_DATA.TEST, BGCON_CNTT_DATA.TRAIN, "ETRS_YM_YEAR", "CLLT_FP_PRNO", 0)
BGCON_CNTT_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TRAIN, BGCON_FPINFO_DATA.TEST, BGCON_CNTT_DATA.TRAIN, "FIRE_YM_YEAR", "CLLT_FP_PRNO", 0)


## BGCON_CLAIM_DATA에서 병명 앞글자로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "CAUS_CODE_FIRST", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "RESL_CD1_FIRST", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 FP 변경 여부(CHANG_FP_YN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "CHANG_FP_YN", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 현재진행구분(CRNT_PROG_DVSN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "CRNT_PROG_DVSN", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 사고구분(ACCI_DVSN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "ACCI_DVSN", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 청구사유코드(DMND_RESN_CODE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "DMND_RESN_CODE", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 유의병원여부(ACCI_HOSP_ADDR)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "ACCI_HOSP_ADDR", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 병원종별구분(HOSP_SPEC_DVSN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "HOSP_SPEC_DVSN", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 유의병원여부(HEED_HOSP_YN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TRAIN, BGCON_CLAIM_DATA.TEST, BGCON_CUST_DATA.TRAIN, "HEED_HOSP_YN", "CUST_ID", 0)

## CLAIM 건수별 유의병원여부(VLID_HOSP_OTDA_TOTAL) = 총 유의병원여부 / 보험처리건
BGCON_CUST_DATA.TRAIN$HEED_HOSP_YN_Y_DIV_BY_CLAIM <- BGCON_CUST_DATA.TRAIN$HEED_HOSP_YN_Y / BGCON_CUST_DATA.TRAIN$CLAIM_TOTAL


## FMLY에서 각 연관 아이디별로 보험사기 여부
BGCON_CUST_DATA.TRAIN.TEMP <- BGCON_CUST_DATA.TRAIN[, c("CUST_ID", "SIU_CUST_YN")]
colnames(BGCON_CUST_DATA.TRAIN.TEMP) <- c("SUB_CUST_ID", "FMLY_SIU_CUST_YN")
MERGE.TEMP.TRAIN <- merge(x=BGCON_FMLY_DATA.TRAIN, y=BGCON_CUST_DATA.TRAIN.TEMP, by = "SUB_CUST_ID", all.x=TRUE)
## FMLY 데이터에 대하여 각 Column을 생성
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(MERGE.TEMP.TRAIN, MERGE.TEMP.TRAIN, BGCON_CUST_DATA.TRAIN, "FMLY_SIU_CUST_YN", "CUST_ID", 0)
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(MERGE.TEMP.TRAIN, MERGE.TEMP.TRAIN, BGCON_CUST_DATA.TRAIN, "FMLY_RELN_CODE", "CUST_ID", 0)



## BGCON_CNTT_DATA에서 고객역할코드(CUST_ROLE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "CUST_ROLE", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 상품분류(GOOD_CLSF_CDNM)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
#BGCON_CNTT_DATA.TRAIN$GOOD_CLSF_CDNM <- factor(as.numeric(factor(BGCON_CNTT_DATA.TRAIN$GOOD_CLSF_CDNM))) #### 한글변수 테스트
#BGCON_CNTT_DATA.TEST$GOOD_CLSF_CDNM <- factor(as.numeric(factor(BGCON_CNTT_DATA.TEST$GOOD_CLSF_CDNM)))  #### 한글변수 테스트
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "GOOD_CLSF_CDNM", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 판매채널코(SALE_CHNL_CODE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "SALE_CHNL_CODE", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 계약상태코드(CNTT_STAT_CODE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "CNTT_STAT_CODE", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 납입주기코드(PAYM_CYCL_CODE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TRAIN, BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TRAIN, "PAYM_CYCL_CODE", "CUST_ID", 0)



# 신용도
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CUST_DATA.TRAIN, BGCON_CUST_DATA.TEST, BGCON_CUST_DATA.TRAIN, "MAXCRDT", "CUST_ID", 0)

# 최소 신용도
BGCON_CUST_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_CUST_DATA.TRAIN, BGCON_CUST_DATA.TEST, BGCON_CUST_DATA.TRAIN, "MINCRDT", "CUST_ID", 0)




## BGCON_FPINFO_DATA 재직구분(INCB_DVSN)데이터를 하나의 컬럼으로 만들어 BGCON_CNTT_DATA merge함
BGCON_CNTT_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TRAIN, BGCON_FPINFO_DATA.TEST, BGCON_CNTT_DATA.TRAIN, "INCB_DVSN", "CLLT_FP_PRNO", 0)

## BGCON_FPINFO_DATA 최종학력(EDGB)데이터를 하나의 컬럼으로 만들어 BGCON_CNTT_DATA merge함
#BGCON_FPINFO_DATA.TRAIN$EDGB <- factor(as.numeric(BGCON_FPINFO_DATA.TRAIN$EDGB)) #### 한글변수 테스트
#BGCON_FPINFO_DATA.TEST$EDGB <- factor(as.numeric(BGCON_FPINFO_DATA.TEST$EDGB)) #### 한글변수 테스트
BGCON_CNTT_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TRAIN, BGCON_FPINFO_DATA.TEST, BGCON_CNTT_DATA.TRAIN, "EDGB", "CLLT_FP_PRNO", 0)

## BGCON_FPINFO_DATA 입사전직업(BEFO_JOB)데이터를 하나의 컬럼으로 만들어 BGCON_CNTT_DATA merge함
#BGCON_FPINFO_DATA.TRAIN$BEFO_JOB <- factor(as.numeric(BGCON_FPINFO_DATA.TRAIN$BEFO_JOB)) #### 한글변수 테스트
BGCON_CNTT_DATA.TRAIN <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TRAIN, BGCON_FPINFO_DATA.TEST, BGCON_CNTT_DATA.TRAIN, "BEFO_JOB", "CLLT_FP_PRNO", 0)


# 테스트중
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "CLLT_FP_PRNO", "CLLT_FP_TOTAL", is.not.na, 0)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "INCB_DVSN_", "INCB_DVSN__TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "INCB_DVSN_#", "INCB_DVSN_SHAP_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "INCB_DVSN_C", "INCB_DVSN_C_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "INCB_DVSN_D", "INCB_DVSN_D_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "INCB_DVSN_P", "INCB_DVSN_P_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "INCB_DVSN_R", "INCB_DVSN_R_TOTAL", mean)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "EDGB_", "EDGB__TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "EDGB_#", "EDGB_SHAP_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "EDGB_고졸", "EDGB_고졸_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "EDGB_대학원졸", "EDGB_대학원졸_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "EDGB_대학졸", "EDGB_대학졸_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "EDGB_전문대졸", "EDGB_전문대졸_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "EDGB_중졸", "EDGB_중졸_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "EDGB_초등졸", "EDGB_초등졸_TOTAL", mean)

BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_", "BEFO_JOB__TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_#", "BEFO_JOB_SHAP_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_경찰", "BEFO_JOB_경찰_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_공무원", "BEFO_JOB_공무원_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_교육공무원", "BEFO_JOB_교육공무원_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_교육부문", "BEFO_JOB_교육부문_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_국영기업체", "BEFO_JOB_국영기업체_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_군인", "BEFO_JOB_군인_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_금융부문", "BEFO_JOB_금융부문_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_기타", "BEFO_JOB_기타_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_농수축산업", "BEFO_JOB_농수축산업_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_무직", "BEFO_JOB_무직_TOTAL", sum)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_보험관계인", "BEFO_JOB_보험관계인_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_사기업체", "BEFO_JOB_사기업체_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_영업직(판", "BEFO_JOB_영업직_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_의료부문", "BEFO_JOB_의료부문_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_일반공무원", "BEFO_JOB_일반공무원_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_일반기업", "BEFO_JOB_일반기업_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_자유업", "BEFO_JOB_자유업_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_자영업", "BEFO_JOB_자영업_TOTAL", mean)
BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TRAIN, "CUST_ID", "BEFO_JOB_학생", "BEFO_JOB_학생_TOTAL", mean)




## BGCON_FPINFO_DATA 각 연관 아이디별로 보험사기 여부
#BGCON_CUST_DATA.FOR.FP.SIU <- train.batch
#BGCON_CNTT_DATA.FOR.FP.SIU <- BGCON_CNTT_DATA[BGCON_CNTT_DATA$CUST_ID %in% BGCON_CUST_DATA.FOR.FP.SIU$CUST_ID, ] 

#BGCON_CUST_DATA.FOR.FP.SIU.TEMP <- BGCON_CUST_DATA.FOR.FP.SIU[, c("CUST_ID", "SIU_CUST_YN")]
#BGCON_CNTT_DATA.FOR.FP.SIU.TEMP <- BGCON_CNTT_DATA.FOR.FP.SIU[, c("CUST_ID", "CLLT_FP_PRNO")]
#MERGE.TEMP <- merge(x=BGCON_CNTT_DATA.FOR.FP.SIU.TEMP, y=BGCON_CUST_DATA.FOR.FP.SIU.TEMP, by = "CUST_ID", all=TRUE)

## BGCON_FPINFO_DATA 데이터에 대하여 각 Column을 생성
#BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP <- add_count_of_value_by_colname(MERGE.TEMP, MERGE.TEMP, BGCON_FPINFO_DATA, "SIU_CUST_YN", "CLLT_FP_PRNO", 0)

## BGCON_FPINFO_DATA FP 보험 사기자 보험 가입 여부 비율 계산
#BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP$SIU_CUST_YN_DIV <- (BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP$SIU_CUST_YN_Y + 1) / (BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP$SIU_CUST_YN_N + 1)

#
#BGCON_CNTT_DATA.FOR.FP.SIU <- merge(x=BGCON_CNTT_DATA, y=BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP, by = "CLLT_FP_PRNO", all=TRUE)

################################################
#BGCON_CNTT_DATA.FOR.FP.SIU <- BGCON_CNTT_DATA.FOR.FP.SIU[BGCON_CNTT_DATA.FOR.FP.SIU$CUST_ID %in% #BGCON_CUST_DATA.TRAIN$CUST_ID, ] 
#BGCON_CUST_DATA.TRAIN <- merge_after_aggregate(BGCON_CNTT_DATA.FOR.FP.SIU, BGCON_CUST_DATA.TRAIN, "CUST_ID", "SIU_CUST_YN_DIV", "FP_SIU_CUST_YN_DIV_MEDIAN", median, 0)






############################# TEST ###################################
## BGCON_CUST_DATA의  YEAR로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CUST_DATA.TEST, BGCON_CUST_DATA.TRAIN, BGCON_CUST_DATA.TEST, "CUST_RGST_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CUST_DATA.TEST, BGCON_CUST_DATA.TRAIN, BGCON_CUST_DATA.TEST, "MAX_PAYM_YM_YEAR", "CUST_ID", 0)

## BGCON_CLAIM_DATA에서 YEAR로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "RECP_DATE_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "ORIG_RESN_DATE_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "RESN_DATE_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "PAYM_DATE_YEAR", "CUST_ID", 0)

## BGCON_CNTT_DATA.TEST에서 YEAR로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "CNTT_YM_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "EXPR_YM_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "EXTN_YM_YEAR", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "LAPS_YM_YEAR", "CUST_ID", 0)

## BGCON_FPINFO_DATA에서 YEAR로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CNTT_DATA에 merge함
BGCON_CNTT_DATA.TEST <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TEST, BGCON_FPINFO_DATA.TRAIN, BGCON_CNTT_DATA.TEST, "ETRS_YM_YEAR", "CLLT_FP_PRNO", 0)
BGCON_CNTT_DATA.TEST <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TEST, BGCON_FPINFO_DATA.TRAIN, BGCON_CNTT_DATA.TEST, "FIRE_YM_YEAR", "CLLT_FP_PRNO", 0)


## BGCON_CLAIM_DATA에서 병명 앞글자로 추출한 데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "CAUS_CODE_FIRST", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "RESL_CD1_FIRST", "CUST_ID", 0)

## BGCON_CLAIM_DATA.TEST에서 FP 변경 여부(CHANG_FP_YN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "CHANG_FP_YN", "CUST_ID", 0)

## BGCON_CLAIM_DATA.TEST에서 현재진행구분(CRNT_PROG_DVSN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "CRNT_PROG_DVSN", "CUST_ID", 0)

## BGCON_CLAIM_DATA.TEST에서 사고구분(ACCI_DVSN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "ACCI_DVSN", "CUST_ID", 0)

## BGCON_CLAIM_DATA.TEST에서 청구사유코드(DMND_RESN_CODE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "DMND_RESN_CODE", "CUST_ID", 0)

## BGCON_CLAIM_DATA.TEST에서 유의병원여부(ACCI_HOSP_ADDR)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "ACCI_HOSP_ADDR", "CUST_ID", 0)

## BGCON_CLAIM_DATA.TEST에서 병원종별구분(HOSP_SPEC_DVSN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "HOSP_SPEC_DVSN", "CUST_ID", 0)

## BGCON_CLAIM_DATA.TEST에서 유의병원여부(HEED_HOSP_YN)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CLAIM_DATA.TEST, BGCON_CLAIM_DATA.TRAIN, BGCON_CUST_DATA.TEST, "HEED_HOSP_YN", "CUST_ID", 0)

## CLAIM 건수별 유의병원여부(VLID_HOSP_OTDA_TOTAL) = 총 유의병원여부 / 보험처리건
BGCON_CUST_DATA.TEST$HEED_HOSP_YN_Y_DIV_BY_CLAIM <- BGCON_CUST_DATA.TEST$HEED_HOSP_YN_Y / BGCON_CUST_DATA.TEST$CLAIM_TOTAL


## FMLY에서 각 연관 아이디별로 보험사기 여부
BGCON_CUST_DATA.TEST.TEMP <- BGCON_CUST_DATA.TEST[, c("CUST_ID", "SIU_CUST_YN")]
colnames(BGCON_CUST_DATA.TEST.TEMP) <- c("SUB_CUST_ID", "FMLY_SIU_CUST_YN")
MERGE.TEMP.TEST <- merge(x=BGCON_FMLY_DATA.TEST, y=BGCON_CUST_DATA.TEST.TEMP, by = "SUB_CUST_ID", all.x=TRUE)
## FMLY 데이터에 대하여 각 Column을 생성
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(MERGE.TEMP.TEST, MERGE.TEMP.TRAIN, BGCON_CUST_DATA.TEST, "FMLY_SIU_CUST_YN", "CUST_ID", 0)
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(MERGE.TEMP.TEST, MERGE.TEMP.TRAIN, BGCON_CUST_DATA.TEST, "FMLY_RELN_CODE", "CUST_ID", 0)



## BGCON_CNTT_DATA에서 고객역할코드(CUST_ROLE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "CUST_ROLE", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 상품분류(GOOD_CLSF_CDNM)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
#BGCON_CNTT_DATA.TEST$GOOD_CLSF_CDNM <- factor(as.numeric(factor(BGCON_CNTT_DATA.TEST$GOOD_CLSF_CDNM)))
#BGCON_CNTT_DATA.TRAIN$GOOD_CLSF_CDNM <- factor(as.numeric(factor(BGCON_CNTT_DATA.TRAIN$GOOD_CLSF_CDNM)))
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "GOOD_CLSF_CDNM", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 판매채널코(SALE_CHNL_CODE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "SALE_CHNL_CODE", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 계약상태코드(CNTT_STAT_CODE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "CNTT_STAT_CODE", "CUST_ID", 0)

## BGCON_CNTT_DATA에서 납입주기코드(PAYM_CYCL_CODE)데이터를 하나의 컬럼으로 만들어 BGCON_CUST_DATA.TEST에 merge함
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CNTT_DATA.TEST, BGCON_CNTT_DATA.TRAIN, BGCON_CUST_DATA.TEST, "PAYM_CYCL_CODE", "CUST_ID", 0)



# 신용도
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CUST_DATA.TEST, BGCON_CUST_DATA.TRAIN, BGCON_CUST_DATA.TEST, "MAXCRDT", "CUST_ID", 0)

# 최소 신용도
BGCON_CUST_DATA.TEST <- add_count_of_value_by_colname(BGCON_CUST_DATA.TEST, BGCON_CUST_DATA.TRAIN, BGCON_CUST_DATA.TEST, "MINCRDT", "CUST_ID", 0)




## BGCON_FPINFO_DATA 재직구분(INCB_DVSN)데이터를 하나의 컬럼으로 만들어 BGCON_CNTT_DATA merge함
BGCON_CNTT_DATA.TEST <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TEST, BGCON_FPINFO_DATA.TRAIN, BGCON_CNTT_DATA.TEST, "INCB_DVSN", "CLLT_FP_PRNO", 0)

## BGCON_FPINFO_DATA 최종학력(EDGB)데이터를 하나의 컬럼으로 만들어 BGCON_CNTT_DATA merge함
#BGCON_FPINFO_DATA.TEST$EDGB <- factor(as.numeric(BGCON_FPINFO_DATA.TEST$EDGB))
#BGCON_FPINFO_DATA.TRAIN$EDGB <- factor(as.numeric(BGCON_FPINFO_DATA.TRAIN$EDGB))
BGCON_CNTT_DATA.TEST <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TEST, BGCON_FPINFO_DATA.TRAIN, BGCON_CNTT_DATA.TEST, "EDGB", "CLLT_FP_PRNO", 0)

## BGCON_FPINFO_DATA 입사전직업(BEFO_JOB)데이터를 하나의 컬럼으로 만들어 BGCON_CNTT_DATA merge함
#BGCON_FPINFO_DATA.TEST$BEFO_JOB <- factor(as.numeric(BGCON_FPINFO_DATA.TEST$BEFO_JOB))
#BGCON_FPINFO_DATA.TRAIN$BEFO_JOB <- factor(as.numeric(BGCON_FPINFO_DATA.TRAIN$BEFO_JOB))
BGCON_CNTT_DATA.TEST <- add_count_of_value_by_colname(BGCON_FPINFO_DATA.TEST, BGCON_FPINFO_DATA.TRAIN, BGCON_CNTT_DATA.TEST, "BEFO_JOB", "CLLT_FP_PRNO", 0)



# 테스트중
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "CLLT_FP_PRNO", "CLLT_FP_TOTAL", is.not.na, 0)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "INCB_DVSN_", "INCB_DVSN__TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "INCB_DVSN_#", "INCB_DVSN_SHAP_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "INCB_DVSN_C", "INCB_DVSN_C_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "INCB_DVSN_D", "INCB_DVSN_D_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "INCB_DVSN_P", "INCB_DVSN_P_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "INCB_DVSN_R", "INCB_DVSN_R_TOTAL", mean)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "EDGB_", "EDGB__TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "EDGB_#", "EDGB_SHAP_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "EDGB_고졸", "EDGB_고졸_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "EDGB_대학원졸", "EDGB_대학원졸_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "EDGB_대학졸", "EDGB_대학졸_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "EDGB_전문대졸", "EDGB_전문대졸_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "EDGB_중졸", "EDGB_중졸_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "EDGB_초등졸", "EDGB_초등졸_TOTAL", mean)

BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_", "BEFO_JOB__TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_#", "BEFO_JOB_SHAP_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_경찰", "BEFO_JOB_경찰_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_공무원", "BEFO_JOB_공무원_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_교육공무원", "BEFO_JOB_교육공무원_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_교육부문", "BEFO_JOB_교육부문_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_국영기업체", "BEFO_JOB_국영기업체_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_군인", "BEFO_JOB_군인_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_금융부문", "BEFO_JOB_금융부문_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_기타", "BEFO_JOB_기타_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_농수축산업", "BEFO_JOB_농수축산업_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_무직", "BEFO_JOB_무직_TOTAL", sum)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_보험관계인", "BEFO_JOB_보험관계인_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_사기업체", "BEFO_JOB_사기업체_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_영업직(판", "BEFO_JOB_영업직_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_의료부문", "BEFO_JOB_의료부문_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_일반공무원", "BEFO_JOB_일반공무원_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_일반기업", "BEFO_JOB_일반기업_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_자유업", "BEFO_JOB_자유업_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_자영업", "BEFO_JOB_자영업_TOTAL", mean)
BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.TEST, BGCON_CUST_DATA.TEST, "CUST_ID", "BEFO_JOB_학생", "BEFO_JOB_학생_TOTAL", mean)




##### Test Data 에 맞는 처리 방법 필요
## BGCON_FPINFO_DATA 각 연관 아이디별로 보험사기 여부
#BGCON_CUST_DATA.FOR.FP.SIU <- train.batch
#BGCON_CNTT_DATA.FOR.FP.SIU <- BGCON_CNTT_DATA[BGCON_CNTT_DATA$CUST_ID %in% BGCON_CUST_DATA.FOR.FP.SIU$CUST_ID, ] 
#BGCON_CUST_DATA.FOR.FP.SIU.TEMP <- BGCON_CUST_DATA.FOR.FP.SIU[, c("CUST_ID", "SIU_CUST_YN")]
#BGCON_CNTT_DATA.FOR.FP.SIU.TEMP <- BGCON_CNTT_DATA.FOR.FP.SIU[, c("CUST_ID", "CLLT_FP_PRNO")]
#MERGE.TEMP <- merge(x=BGCON_CNTT_DATA.FOR.FP.SIU.TEMP, y=BGCON_CUST_DATA.FOR.FP.SIU.TEMP, by = "CUST_ID", all=TRUE)
## BGCON_FPINFO_DATA 데이터에 대하여 각 Column을 생성
#BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP <- add_count_of_value_by_colname(MERGE.TEMP, MERGE.TEMP, BGCON_FPINFO_DATA, "SIU_CUST_YN", "CLLT_FP_PRNO", 0)
## BGCON_FPINFO_DATA FP 보험 사기자 보험 가입 여부 비율 계산
#BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP$SIU_CUST_YN_DIV <- (BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP$SIU_CUST_YN_Y + 1) / (BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP$SIU_CUST_YN_N + 1)
#
#BGCON_CNTT_DATA.FOR.FP.SIU <- merge(x=BGCON_CNTT_DATA, y=BGCON_FPINFO_DATA.FOR.FP.SIU.TEMP, by = "CLLT_FP_PRNO", all=TRUE)
################################################
#BGCON_CNTT_DATA.FOR.FP.SIU <- BGCON_CNTT_DATA.FOR.FP.SIU[BGCON_CNTT_DATA.FOR.FP.SIU$CUST_ID %in% #BGCON_CUST_DATA.TEST$CUST_ID, ] 
#BGCON_CUST_DATA.TEST <- merge_after_aggregate(BGCON_CNTT_DATA.FOR.FP.SIU, BGCON_CUST_DATA.TEST, "CUST_ID", "SIU_CUST_YN_DIV", "FP_SIU_CUST_YN_DIV_MEDIAN", median, 0)



#######################################################################
############################ WRITE RESULT #############################
#######################################################################
# 처리된 데이터 csv 파일로 저장
#write.csv(BGCON_CLAIM_DATA.TRAIN, "data/BGCON_CLAIM_DATA_TRAIN.csv")
#write.csv(BGCON_CNTT_DATA.TRAIN, "data/BGCON_CNTT_DATA_TRAIN.csv")
#write.csv(BGCON_CUST_DATA.TRAIN, "data/BGCON_CUST_DATA_TRAIN.csv")
#write.csv(BGCON_FMLY_DATA.TRAIN, "data/BGCON_FMLY_DATA_TRAIN.csv")
#write.csv(BGCON_FPINFO_DATA.TRAIN, "data/BGCON_FPINFO_DATA_TRAIN.csv")

#write.csv(BGCON_CLAIM_DATA.TEST, "data/BGCON_CLAIM_DATA_TEST.csv")
#write.csv(BGCON_CNTT_DATA.TEST, "data/BGCON_CNTT_DATA_TEST.csv")
#write.csv(BGCON_CUST_DATA.TEST, "data/BGCON_CUST_DATA_TEST.csv")
#write.csv(BGCON_FMLY_DATA.TEST, "data/BGCON_FMLY_DATA_TEST.csv")
#write.csv(BGCON_FPINFO_DATA.TEST, "data/BGCON_FPINFO_DATA_TEST.csv")




#######################################################################
########################### MISSING VALUE #############################
#######################################################################
print_missing_count <- function(df, all = FALSE){
   num_data <- nrow(df)
   for (name in colnames(df)){
       sum_col <- sum(is.na(df[, name]))
       if (all == TRUE || sum_col > 0) {
           cat(name, "(", class(df[, name]), ")" , "====", sum_col, "\t\trate = ", round(sum_col/num_data*100, 1))
           cat("\n")
       }
   }
}
remove_missing_data_column <- function(df, threshold = 100){
   num_data <- nrow(df)
   for (name in colnames(df)){
       sum_mis_row <- sum(is.na(df[, name]))
       if (sum_mis_row > 0) {
         rate <- round(sum_mis_row/num_data*100, 1)
         cat(name, "(", class(df[, name]), ")" , "====", sum_mis_row, "\t\trate = ", rate)
         cat("\n")
         if(rate > threshold) {
           df[, name] <- NULL
         }
       }
   }
  return(df)
}
remove_missing_data_row <- function(df){
   num_data <- nrow(df)
   for (name in colnames(df)){
     sum_mis_row <- sum(is.na(df[, name]))
     if (sum_mis_row > 0) {
       df[is.na(df[, name]),] <- NULL
     }
   }
  return(df)
}

colnames(BGCON_CUST_DATA.TRAIN)
colnames(BGCON_CUST_DATA.TEST)
BGCON_CUST_DATA.TEST$FMLY_SIU_CUST_YN_ <- NULL

BGCON_CUST_DATA.TRAIN.MTEST <- BGCON_CUST_DATA.TRAIN
BGCON_CUST_DATA.TEST.MTEST <- BGCON_CUST_DATA.TEST

BGCON_CUST_DATA.TRAIN.MTEST$SET <- 1 #TRAIN
BGCON_CUST_DATA.TEST.MTEST$SET <- 2 #TEST

BGCON_CUST_DATA.MISS.SET <- rbind(BGCON_CUST_DATA.TRAIN.MTEST, BGCON_CUST_DATA.TEST.MTEST)
print_missing_count(BGCON_CUST_DATA)

###살려야할 결측치 변수들
# RESI_TYPE_CODE  거주TYPE
# CHLD_CNT  자녀수
# LTBN_CHLD_AGE  만내 자녀 연령
# RCBASE_HSHD_INCM  추정가구소득1	고객의 주택가격을 우선하여 정한 가구소득 추정금액
# JPBASE_HSHD_INCM	추정가구소득2	고객의 직업 및 납입보험료 수준을 우선하여 정한 가구소득 추정금액
# MAIN_INSR_AMT_TOTAL 주보험금	주계약의 보험금액(단위:원)
# MAIN_INSR_AMT_MEAN 
# MAIN_INSR_AMT_MEDIAN
# SUM_ORIG_PREM_TOTAL 합계보험료	계약(주계약 + 특약)의 전체 보험료
# SUM_ORIG_PREM_MEAN
# SUM_ORIG_PREM_MEDIAN
# MNTH_INCM_AMT_TOTAL
# MNTH_INCM_AMT_MEAN 청약서 소득	청약서에 기재한 월소득금액
# MNTH_INCM_AMT_MEDIAN 청약서 소득	청약서에 기재한 월소득금액
#### df missing column 제거
#test.df <- remove_missing_data_column(BGCON_CUST_DATA.MISS.SET, 10)
#print_missing_count(test.df)

#impute missing values, using all parameters as default values
#test.df.imp <- missForest(test.df)

#check imputed values
#test.df.imp$ximp

#check imputation error
#test.df.imp$OOBerror

#str(test.df.imp)
#write.csv(test.df.imp$ximp, "data/MISSING_FILL_DATA.csv")

#INPUT <- test.df.imp$ximp$SIU_CUST_YN
#BGCON_CUST_DATA.TRAIN.MS <- test.df.imp$ximp[which(test.df.imp$ximp$SET == 1), ]
#BGCON_CUST_DATA.TEST.MS <- test.df.imp$ximp[which(test.df.imp$ximp$SET == 2), ]


######## Missing Value채우않고 경우
BGCON_CUST_DATA.MISS.SET <- remove_missing_data_column(BGCON_CUST_DATA.MISS.SET, 0)
BGCON_CUST_DATA.TRAIN.MS <- BGCON_CUST_DATA.MISS.SET[which(BGCON_CUST_DATA.MISS.SET$SET == 1), ]
BGCON_CUST_DATA.TEST.MS <- BGCON_CUST_DATA.MISS.SET[which(BGCON_CUST_DATA.MISS.SET$SET == 2), ]




#######################################################################
########################## MODEL TRAIN PRE ############################
#######################################################################

###################### Model Test용으로 복사
set.seed(13784)
#trainningData <- downSample(x = BGCON_CUST_DATA.TRAIN.MS, y = BGCON_CUST_DATA.TRAIN.MS$SIU_CUST_YN)
trainningData <- BGCON_CUST_DATA.TRAIN.MS
testingData <- BGCON_CUST_DATA.TEST.MS

###################### 앙상블로 데이터 분리
#set.seed(23)
#ensemble.rows <- createDataPartition(BGCON_CUST_DATA.TEST.MODEL$SIU_CUST_YN,  p = 0.5, list = FALSE)
#blenderData <- BGCON_CUST_DATA.TEST.MODEL[ensemble.rows, ]
#testingData <- BGCON_CUST_DATA.TEST.MODEL[-ensemble.rows, ]

####################### Train 시 영향을 주는 정답 변수 제거
A.BGCON_CUST_DATA.TRAIN.MODEL <- BGCON_CUST_DATA.TRAIN.MS
A.BGCON_CUST_DATA.TEST.MODEL <- BGCON_CUST_DATA.TEST.MS

A.BGCON_CUST_DATA.TRAIN.MODEL$SIU_CUST_YN <- as.numeric(A.BGCON_CUST_DATA.TRAIN.MODEL$SIU_CUST_YN)
A.BGCON_CUST_DATA.TRAIN.MODEL$FP_CAREER <- as.numeric(A.BGCON_CUST_DATA.TRAIN.MODEL$FP_CAREER)
A.BGCON_CUST_DATA.TRAIN.MODEL$CTPR <- as.numeric(A.BGCON_CUST_DATA.TRAIN.MODEL$CTPR)
A.BGCON_CUST_DATA.TRAIN.MODEL$OCCP_GRP_1 <- as.numeric(A.BGCON_CUST_DATA.TRAIN.MODEL$OCCP_GRP_1)
A.BGCON_CUST_DATA.TRAIN.MODEL$OCCP_GRP_2 <- as.numeric(A.BGCON_CUST_DATA.TRAIN.MODEL$OCCP_GRP_2)
A.BGCON_CUST_DATA.TRAIN.MODEL$MATE_OCCP_GRP_1 <- as.numeric(A.BGCON_CUST_DATA.TRAIN.MODEL$MATE_OCCP_GRP_1)
A.BGCON_CUST_DATA.TRAIN.MODEL$MATE_OCCP_GRP_2 <- as.numeric(A.BGCON_CUST_DATA.TRAIN.MODEL$MATE_OCCP_GRP_2)
A.BGCON_CUST_DATA.TRAIN.MODEL$WEDD_YN <- as.numeric(A.BGCON_CUST_DATA.TRAIN.MODEL$WEDD_YN)
A.BGCON_CUST_DATA.TRAIN.MODEL$SET <- NULL

A.BGCON_CUST_DATA.TEST.MODEL$SIU_CUST_YN <- as.numeric(A.BGCON_CUST_DATA.TEST.MODEL$SIU_CUST_YN)
A.BGCON_CUST_DATA.TEST.MODEL$FP_CAREER <- as.numeric(A.BGCON_CUST_DATA.TEST.MODEL$FP_CAREER)
A.BGCON_CUST_DATA.TEST.MODEL$CTPR <- as.numeric(A.BGCON_CUST_DATA.TEST.MODEL$CTPR)
A.BGCON_CUST_DATA.TEST.MODEL$OCCP_GRP_1 <- as.numeric(A.BGCON_CUST_DATA.TEST.MODEL$OCCP_GRP_1)
A.BGCON_CUST_DATA.TEST.MODEL$OCCP_GRP_2 <- as.numeric(A.BGCON_CUST_DATA.TEST.MODEL$OCCP_GRP_2)
A.BGCON_CUST_DATA.TEST.MODEL$MATE_OCCP_GRP_1 <- as.numeric(A.BGCON_CUST_DATA.TEST.MODEL$MATE_OCCP_GRP_1)
A.BGCON_CUST_DATA.TEST.MODEL$MATE_OCCP_GRP_2 <- as.numeric(A.BGCON_CUST_DATA.TEST.MODEL$MATE_OCCP_GRP_2)
A.BGCON_CUST_DATA.TEST.MODEL$WEDD_YN <- as.numeric(A.BGCON_CUST_DATA.TEST.MODEL$WEDD_YN)
A.BGCON_CUST_DATA.TEST.MODEL$SET <- NULL



#######################################################################
############################ SMOTE RESULT #############################
#######################################################################

set.seed(13784)
trainningData.smote <- SMOTE(SIU_CUST_YN ~ . , trainningData, perc.over = 200, perc.under = 300)
#nrow(trainningData.smote); nrow(trainningData.smote[which(trainningData.smote$SIU_CUST_YN == "Y"),]);


trainningData.smote$SIU_CUST_YN <- factor(trainningData.smote$SIU_CUST_YN)
trainningData.smote$FP_CAREER <- as.numeric(trainningData.smote$FP_CAREER)
trainningData.smote$CTPR <- as.numeric(trainningData.smote$CTPR)
trainningData.smote$OCCP_GRP_1 <- as.numeric(trainningData.smote$OCCP_GRP_1)
trainningData.smote$OCCP_GRP_2 <- as.numeric(trainningData.smote$OCCP_GRP_2)
trainningData.smote$MATE_OCCP_GRP_1 <- as.numeric(trainningData.smote$MATE_OCCP_GRP_1)
trainningData.smote$MATE_OCCP_GRP_2 <- as.numeric(trainningData.smote$MATE_OCCP_GRP_2)
trainningData.smote$WEDD_YN <- as.numeric(trainningData.smote$WEDD_YN)
trainningData.smote$SET <- NULL

testingData$FP_CAREER <- as.numeric(testingData$FP_CAREER)
testingData$CTPR <- as.numeric(testingData$CTPR)
testingData$OCCP_GRP_1 <- as.numeric(testingData$OCCP_GRP_1)
testingData$OCCP_GRP_2 <- as.numeric(testingData$OCCP_GRP_2)
testingData$MATE_OCCP_GRP_1 <- as.numeric(testingData$MATE_OCCP_GRP_1)
testingData$MATE_OCCP_GRP_2 <- as.numeric(testingData$MATE_OCCP_GRP_2)
testingData$WEDD_YN <- as.numeric(testingData$WEDD_YN)
testingData$SET <- NULL



#######################################################################
######################### Feature Importance ##########################
#######################################################################

#rf.feature.importance <- randomForest(SIU_CUST_YN ~ + ., data = trainningData.smote, importance = TRUE)
#importance(rf.feature.importance)
#varImpPlot(rf.feature.importance, main = "varImpPlot")
#rf.feature.importance$importance[order(rf.feature.importance$importance[, 3], decreasing=FALSE),]


#######################################################################
############################ TRAIN MODEL ##############################
#######################################################################

# Set up training control
set.seed(13784)
cv.ctrl <- trainControl(method = "repeatedcv",   # 10fold cross validation
                        number = 3,							# do 5 repititions of cv
                        summaryFunction = twoClassSummary,	# Use AUC to pick the best model
                        classProbs=TRUE,
                        allowParallel = TRUE)

xgb.grid <- expand.grid(nrounds = c(50, 200), #the maximum number of iterations
                        eta = c(0.6), # shrinkage
                        max_depth = c(2, 3),
                        gamma = 0,
                        colsample_bytree = c(0.4, 0.8),    #default=1
                        min_child_weight = 1     #default=1
                        )


model.tune.1 <- train(SIU_CUST_YN ~ VLID_HOSP_OTDA_TOTAL  + VLID_HOSP_OTDA_MEDIAN  + VLID_HOSP_OTDA_MEAN  + DMND_RESN_CODE_2  + PAYM_AMT_TOTAL  + DMND_AMT_TOTAL  + PAYM_AMT_MEAN  + DMND_AMT_MEAN  + CLAIM_TOTAL  + HEED_HOSP_YN_N  + PAYM_AMT_MEDIAN  + DMND_AMT_MEDIAN  + RESL_CD1_FIRST_M  + CRNT_PROG_DVSN_23  + HOSP_SPEC_DVSN_80  + CAUS_CODE_FIRST_M  + ACCI_DVSN_3  + OCCP_GRP_2 + RESN_DATE_YEAR_2010  + PAYM_DATE_YEAR_2010  + HOSP_SPEC_DVSN_30  + RECP_DATE_YEAR_2010  + PAYM_DATE_YEAR_2011  + ORIG_RESN_DATE_YEAR_2010  + CTPR  + HOSP_SPEC_DVSN_95  + RECP_DATE_YEAR_2011  + RESL_CD1_FIRST_S  + ACCI_HOSP_ADDR_  + PAYM_DATE_YEAR_2009  + CRNT_PROG_DVSN_33  + OCCP_GRP_1  + CHANG_FP_YN_N  + RECP_DATE_YEAR_2009  + RESN_DATE_YEAR_2011  + DMND_RESN_CODE_3  + HOSP_SPEC_DVSN_10  + CAUS_CODE_FIRST_W  + RESN_DATE_YEAR_2009  + ACCI_DVSN_1  + AGE  + DMND_RESN_CODE_5  + CHANG_FP_YN_Y  + ORIG_RESN_DATE_YEAR_2009  + ORIG_RESN_DATE_YEAR_2011  + REAL_PAYM_TERM_TOTAL  + SALE_CHNL_CODE_1  + CNTT_STAT_CODE_4  + DMND_RESN_CODE_6  + CNTT_STAT_CODE_1  + ACCI_DVSN_2  + RESN_DATE_YEAR_2012  + CLLT_FP_TOTAL  + PAYM_CYCL_CODE_1  + REAL_PAYM_TERM_MEAN  + CNTT_TOTAL  + PAYM_DATE_YEAR_2012  + RECP_DATE_YEAR_2012  + CAUS_CODE_FIRST_V  + ACCI_HOSP_ADDR_서울  + PAYM_DATE_YEAR_2013  + EXPR_YM_YEAR_9999  + HOSP_SPEC_DVSN_20  + ORIG_RESN_DATE_YEAR_2008  + RESN_DATE_YEAR_2013  + ORIG_RESN_DATE_YEAR_2012  + ORIG_RESN_DATE_YEAR_2013  + MATE_OCCP_GRP_2  + CNTT_STAT_CODE_B  + BEFO_JOB_보험관계인_TOTAL  + RECP_DATE_YEAR_2008  + DMND_RESN_CODE_4  + CUST_ROLE_1  + MATE_OCCP_GRP_1
                    ,data=trainningData.smote
                    ,method = "xgbTree"
                    ,metric = "ROC"
                    #,tuneGrid = xgb.grid
                    ,trControl = cv.ctrl)

model.tune.1; plot(model.tune.1);

#### Model Evaluation
model.pred.1 <- predict(model.tune.1, testingData)
model.prob.1 <- predict(model.tune.1, testingData, type="prob")
(confusion <- (confusionMatrix(model.pred.1, testingData$SIU_CUST_YN)))


# F-Score: 2 * precision(Pos Pred Value) * recall(Sensitivity) /(precision + recall):
f1_score <- (2 * confusion$byClass[3] * confusion$byClass[1]) / (confusion$byClass[3] + confusion$byClass[1])
names(f1_score) <- "F1 Score"
f1_score


#######################################################################
######################### write final result ##########################
#######################################################################

final_result <- cbind(model.pred.1, testingData)
write.csv(final_result, file="final_result.csv")

```


