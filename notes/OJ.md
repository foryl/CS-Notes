#### 计数器
我们经常使用一组整数计数器来描述系统状态。需要在某一时刻保存所有计数器的值，作为系统的快照，并在以后能获得这个快照。系统可能会存在多个快照。

请完成如下功能：

1）调用CreateCounter()创建计数器，计数器ID使用唯一的32位整数进行标识；可以创建多个计数器,这些计数器是全局的

2）调用AddCounterValue()，对指定计数器的计数值加一

3）调用GetCounterValue()，获取指定计数器ID的当前计数值

4）调用GetCounterNum()，获取当前系统中存在的计数器个数

5）调用SaveSnapshot()接口，保存系统内所有计数器的当前计数值，输入参数为快照ID 

6）调用GetSavedSnapshot()接口，获取指定快照ID的某个计数器的计数值

7）调用GetSnapshotNum(), 获取当前系统中存在的快照个数

8）调用DeleteSnapshot(),删除指定的某个快照

9) 调用ClearAll()，删除所有计数器和快照

系统约束

1.计数器ID以及快照ID的取值范围均为0~0xFFFFFFFF

2.计数器被创建后，初始计数值为0 

3.创建ID重复的计数器时，程序应该返回失败

4.查询不存在的计数器ID或者快照ID时，程序应该返回失败

5.对同一快照ID，如果有多次Save操作，则以最后一次保存的值为准

```c
/******************************************************************************

Copyright (C), 2001-2011, Huawei Tech. Co., Ltd.

******************************************************************************
File Name     :
Version       :
Author        :
Created       : 2009/10/10
Last Modified :
Description   :
Function List :

History       :
1.Date        : 2009/10/10
Author      :
Modification: Created file

******************************************************************************/
#include <stdlib.h>
#define N 100

typedef struct {
	unsigned long ulCounterValue;
	unsigned long ulCounterId;
}COUNTER;

typedef struct {
	unsigned long ulSnapshotId;
	COUNTER astCountList[N];
}SNAPSHOT;

unsigned long g_ulCounterNum = 0;
COUNTER *g_apstCounterList[N] = { NULL };
unsigned long g_ulSnapshotNum = 0;
SNAPSHOT *g_apstSnapshotList[N] = { NULL };


/*****************************************************************************
Description    : 创建计数器
Prototype      : int CreateCounter(unsigned long ulCounterId);
Input Param    : unsigned long ulCounterId  创建的计数器ID
Return Value   : 成功返回0，失败返回-1
*****************************************************************************/
int CreateCounter(unsigned long ulCounterId)
{
	for (unsigned long i = 0; i < g_ulCounterNum; i++) {
		/* 计数器ID已存在 */
		if (g_apstCounterList[i]->ulCounterId == ulCounterId) {
			return -1;
		}
	}
	COUNTER *pstNewCounter = (COUNTER*)malloc(sizeof(COUNTER));
	if (pstNewCounter == NULL) {
		return -1;
	}
	pstNewCounter->ulCounterId = ulCounterId;
	pstNewCounter->ulCounterValue = 0;
	g_apstCounterList[g_ulCounterNum++] = pstNewCounter;
	return 0;
}


/*****************************************************************************
Description    : 对指定的计数器ID的计数值加一
Prototype      : int AddCounterValue (unsigned long ulCounterId);
Input Param    : unsigned long ulCounterId  计数器ID
Return Value   : 成功返回0，失败返回-1
*****************************************************************************/
int AddCounterValue(unsigned long ulCounterId)
{
	for (unsigned long i = 0; i < g_ulCounterNum; i++) {
		if (g_apstCounterList[i] == NULL) {
			return -1;
		}
		if (g_apstCounterList[i]->ulCounterId == ulCounterId) {
			g_apstCounterList[i]->ulCounterValue++;
			break;
		}
	}
	return 0;
}

/*****************************************************************************
Description    : 获取指定的计数器ID的当前计数值 .
Prototype      : int GetCounterValue (unsigned long ulCounterId, unsigned long *pValue);
Input Param    : unsigned long ulCounterId    计数器ID
Oueput Param   : unsigned long *pValue        需要查询的计数器ID的当前计数值
Return Value   : 成功返回0，失败返回-1

*****************************************************************************/
int GetCounterValue(unsigned long ulCounterId, unsigned long *pValue)
{
	for (unsigned long i = 0; i < g_ulCounterNum; i++) {
		if (g_apstCounterList[i] == NULL) {
			return -1;
		}
		if (g_apstCounterList[i]->ulCounterId == ulCounterId) {
			*pValue = g_apstCounterList[i]->ulCounterValue;
			break;
		}
	}
	return 0;
}


/*****************************************************************************
Description      : 获取系统中存在的当前计数器个数
Prototype        : int GetCounterNum (unsigned long *pulCounterNum);
Input Param      : 无
Output Param     : unsigned long *pulCounterNum当前系统中存在的计数器个数
Return Value     : 成功返回0，失败返回-1
*****************************************************************************/
int GetCounterNum(unsigned long *pulCounterNum)
{
	*pulCounterNum = g_ulCounterNum;
	return 0;
}


/*****************************************************************************
Description     : 保存系统中所有计数器的当前计数值
Prototype       : int SaveSnapshot(unsigned long ulSnapshotId);
Input Param     : unsigned long ulSnapshotId 快照ID
Return Value    : 成功返回0，失败返回-1

*****************************************************************************/
int SaveSnapshot(unsigned long ulSnapshotId)
{
	SNAPSHOT *pstNewSnapshot = (SNAPSHOT*)malloc(sizeof(SNAPSHOT));
	if (pstNewSnapshot == NULL) {
		return -1;
	}
	pstNewSnapshot->ulSnapshotId = ulSnapshotId;
	for (unsigned long i = 0; i < g_ulCounterNum; i++) {
		pstNewSnapshot->astCountList[i].ulCounterId = g_apstCounterList[i]->ulCounterId;
		pstNewSnapshot->astCountList[i].ulCounterValue = g_apstCounterList[i]->ulCounterValue;
	}
	/* 若快照ID已存在，则更新值 */
	for (unsigned long i = 0; i < N; i++) {
		if (g_apstSnapshotList[i] == NULL) {
			continue;
		}
		if (g_apstSnapshotList[i]->ulSnapshotId == ulSnapshotId) {
			free(g_apstSnapshotList[i]);
			g_apstSnapshotList[i] = pstNewSnapshot;
			return 0;
		}
	}
	/* 新快照添加至快照列表 */
	g_apstSnapshotList[g_ulSnapshotNum++] = pstNewSnapshot;

	return 0;
}


/*****************************************************************************
Description      : 获取指定快照ID的某个计数器的计数值
Prototype        : int GetSavedSnapshot(unsigned long ulSnapshotId, unsigned long ulCounterId, ULONG *pValue);
Input Param      : unsigned long ulSnapshotId   快照ID
unsigned long ulCounterId    计数器ID
Output Param     : unsigned long *pValue        快照ID指定的计数器ID的计数器值
Return Value     :  成功返回0，失败返回-1

*****************************************************************************/
int GetSavedSnapshot(unsigned long ulSnapshotId, unsigned long ulCounterId, unsigned long *pValue)
{
	for (int i = 0; i < N; i++) {
		if (g_apstSnapshotList[i] == NULL) {
			continue;
		}
		/* 查找对应快照ID*/
		if (g_apstSnapshotList[i]->ulSnapshotId == ulSnapshotId) {
			for (unsigned long j = 0; j < g_ulCounterNum; j++) {
				/* 查找对应计数器 */
				if (g_apstSnapshotList[i]->astCountList[j].ulCounterId == ulCounterId) {
					*pValue = g_apstSnapshotList[i]->astCountList[j].ulCounterValue;
					return 0;
				}
			}
		}

	}

	return -1;
}


/*****************************************************************************
Description     : 获取当前系统中存在的快照个数
Prototype       : int GetSnapshotNum(unsigned long *pulSnapshotNum);
Input Param     : 无
Output Param    : unsigned long pulSnapshotNum  快照个数
Return Value    : 成功返回0，失败返回-1
*****************************************************************************/
int GetSnapshotNum(unsigned long *pulSnapshotNum)
{
	*pulSnapshotNum = g_ulSnapshotNum;
	return 0;
}


/*****************************************************************************
Description     : 删除指定的某个快照
Prototype       : int DeleteSnapshot(unsigned long ulSnapshotId);
Input Param     : unsigned long ulSnapshotId  快照ID
Output Param    : 无
Return Value    : 成功返回0，失败返回-1
*****************************************************************************/
int DeleteSnapshot(unsigned long ulSnapshotId)
{
	for (int i = 0; i < N; i++) {
		if (g_apstSnapshotList[i] == NULL) {
			continue;
		}
		/* 牢记释放内存 */
		if (g_apstSnapshotList[i]->ulSnapshotId == ulSnapshotId) {
			free(g_apstSnapshotList[i]);
			g_apstSnapshotList[i] = NULL;
		}
	}
	g_ulSnapshotNum--;
	return 0;
}


/*****************************************************************************
Description     : 删除所有计数器和快照。为了保证测试用例之间没有相关性，每个用例执行结束调用
Prototype       : void ClearAll(void);
Input Param     : 无
Return Value    : 无
*****************************************************************************/
void ClearAll(void)
{
	/* 删除 快照 */
	for (int i = 0; i < N; i++) {
		if (g_apstSnapshotList[i] == NULL) {
			continue;
		}
		free(g_apstSnapshotList[i]);
		g_apstSnapshotList[i] = NULL;
	}
	g_ulSnapshotNum = 0;
	/* 删除 计数器*/
	for (unsigned long i = 0; i < g_ulCounterNum; i++) {
		free(g_apstCounterList[i]);
		g_apstCounterList[i] = NULL;
	}
	g_ulCounterNum = 0;
	return;
}
```


 
