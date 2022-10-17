# 第一题答案：每位医生各收录几名患者？其中各有几名患者上传了血糖？按收录患者数量降序排列
select
	医生id,
    count(患者id) as 收录患者数量,
    sum(是否上传血糖) as 上传血糖患者数量
from
	(select
		t1.pat_id as 患者id,
		t1.doc_id as 医生id,
		if(glucose is not null,1,0) as 是否上传血糖
	from
		patient_list as t1
	left join
		glucose as t2
	on
		t1.pat_id=t2.pat_id) as tbl1
group by
	医生id
order by
	收录患者数量 desc;
  
  
  
  # 第二题答案：不会MongoDB操作，工作中如需用到后续可以学习
  
  
  
  # 第三题答案
  ## 第一组按医生分组分析：
  ### 第一小题：患者年龄分布是什么样的？
  import pandas as pd
import matplotlib.pyplot as plt
plt.rcParams["font.sans-serif"]=["Microsoft YaHei"]
plt.rcParams["axes.unicode_minus"]=False

# 读入数据
patient=pd.read_csv(filepath_or_buffer='patient_list.csv',sep='\t')
glucose=pd.read_csv(filepath_or_buffer='glucose.csv',sep='\t')

# 处理数据
patient_glucose=pd.merge(left=patient,right=glucose,how='left',on='pat_id')
patient_glucose['年龄分组']=pd.cut(x=patient_glucose['age'],bins=[-1,29,39,49,150],
                               labels=['青年组','中青年组','中年组','中老年组'])
patient_glucose_groupby=patient_glucose.groupby(by=['doc_id','年龄分组'])['pat_id'].\
count()
list1=list()
for i in list(patient_glucose_groupby.index.values):
    str1=str()
    for j in i:
        str1+=j
    list1.append(str1)
list1

# 画图并且打印图表
plt.figure(figsize=(8,5),dpi=100)
picture=plt.bar(x=list1,height=patient_glucose_groupby.values,label='value')
plt.bar_label(container=picture)
plt.xticks(rotation=-45)
plt.title(label='患者年龄分布图',
          fontdict={'fontname':'Microsoft YaHei','fontsize':12})
plt.show()



### 第二小题：最近一个月有多少比例的患者上传了血糖数据？上传血糖的患者中，人均测量次数是多少？
import pandas as pd
import matplotlib.pyplot as plt

# 读入数据
patient=pd.read_csv(filepath_or_buffer='patient_list.csv',sep='\t')
glucose=pd.read_csv(filepath_or_buffer='glucose.csv',sep='\t')

# 处理数据
glucose_patient=pd.merge(left=glucose,right=patient,how='left',on='pat_id')
recent_numbers=glucose_patient.drop_duplicates(subset='pat_id').\
groupby(by='doc_id')['pat_id'].count()
all_numbers=patient.shape[0]
ratio=pd.DataFrame(data=recent_numbers).rename({'pat_id':'numbers'},axis=1)
ratio['ratio']=ratio['numbers'].map(lambda x:str(round(number=x/all_numbers*100,
                                                           ndigits=2))+'%')
ratio=ratio[['ratio']]
numbers=glucose_patient.drop_duplicates(subset='pat_id').groupby(by='doc_id')\
['pat_id'].count()
people_numbers=glucose.drop_duplicates(subset='pat_id').shape[0]
average_numbers=pd.DataFrame(data=numbers).rename({'pat_id':'numbers'},axis=1)
average_numbers['average_numbers']=average_numbers['numbers'].\
map(lambda x:round(number=x/people_numbers,ndigits=2))
average_numbers=average_numbers[['average_numbers']]

# 输出结果
print('最近一个月上传血糖数据患者比例：',ratio,sep='\n',end='\n\n\n')
print('最近一个月人均测量次数：',average_numbers,sep='\n')



### 第三小题：测量次数>=8次的患者各有多少人，这些患者的人均测量次数是多少？
import pandas as pd

# 读入数据
patient=pd.read_csv(filepath_or_buffer='patient_list.csv',sep='\t')
glucose=pd.read_csv(filepath_or_buffer='glucose.csv',sep='\t')

# 处理数据
glucose_patient=pd.merge(left=glucose,right=patient,how='left',on='pat_id')
measure_numbers=glucose_patient.groupby(by=['doc_id','pat_id'])[['pat_id']].\
count().rename({'pat_id':'measure_numbers'},axis=1)
measure_numbers['measure_numbers']=measure_numbers['measure_numbers']\
[measure_numbers['measure_numbers'].map(lambda x:x>=8)]
measure_numbers.dropna(inplace=True)
people_numbers=glucose.drop_duplicates(subset='pat_id').shape[0]
measure_numbers['average_numbers']=measure_numbers['measure_numbers'].\
map(lambda x:round(number=x/people_numbers,ndigits=2))

# 输出结果
print(measure_numbers)



### 第四小题：
#### 从化验数据中找出所有糖化血红蛋白（即HbA1c，默认单位是“%”）数据，分别分析总体患者及每位医生管理的患者中，基线值（即患者收录日期前180天至收录日期后30天之间的最早一次化验）与管理一段时间之后的最新化验结果（即当前日期之前90天内的最后一次化验）的变化：
####    i.管理前后的平均值和/或中位数分别多少，是否有显著差异？
####    ii.设糖化血红蛋白达标为：年龄>=65岁的患者要求 < 7.5%，年龄<65岁的患者要求<7%。不良为>9.5%。达标率和不良率分别如何？管理前后是否有显著差异？
import pandas as pd 
import datetime

# 导入数据
patient=pd.read_csv(filepath_or_buffer='patient_list.csv',sep='\t')
lab=pd.read_csv(filepath_or_buffer='lab.csv',sep='\t')

# 处理数据
patient_lab=pd.merge(left=patient,right=lab,how='left',on='pat_id')
patient_lab['ago_180']=pd.to_datetime(patient_lab['create_time']).\
map(lambda x:x+datetime.timedelta(days=-180))
patient_lab['later_30']=pd.to_datetime(patient_lab['create_time']).\
map(lambda x:x+datetime.timedelta(days=30))
patient_lab['now']='2021-07-01 00:00:00'
patient_lab['ago_90']=(pd.to_datetime(arg='2021-07-01')+\
                       datetime.timedelta(days=-90)).strftime('%Y-%m-%d %H:%M:%S')
patient_lab_180_30=patient_lab[(patient_lab['lab_date']>=patient_lab['ago_180'])&\
                               (patient_lab['lab_date']<=patient_lab['later_30'])]
patient_lab_180_30_min=patient_lab_180_30[patient_lab_180_30['lab_date']==\
                                          patient_lab_180_30['lab_date'].min()]
patient_lab_90_now=patient_lab[(patient_lab['lab_date']>=patient_lab['ago_90'])&\
                               (patient_lab['lab_date']<=patient_lab['now'])]
patient_lab_90_now_max=patient_lab_90_now[patient_lab_90_now['lab_date']==\
                                          patient_lab_90_now['lab_date'].max()]
patient_lab_90_now_max=patient_lab_90_now_max[patient_lab_90_now_max\
                                              ['labitems_name']=='HbA1c']
patient_lab_180_30_min_mean=patient_lab_180_30_min['lab_value'].mean()
patient_lab_180_30_min_median=patient_lab_180_30_min['lab_value'].median()
patient_lab_90_now_max_mean=patient_lab_90_now_max['lab_value'].mean()
patient_lab_90_now_max_median=patient_lab_90_now_max['lab_value'].median()
before_standards=len(patient_lab_180_30_min[((patient_lab_180_30_min['age']>=65)&\
                                      (patient_lab_180_30_min['lab_value']<'7.5'))|\
                                     ((patient_lab_180_30_min['age']<65)&\
                                      (patient_lab_180_30_min['lab_value']<'7'))])
before_bad=len(patient_lab_180_30_min[patient_lab_180_30_min['lab_value']>'9.5'])
after_standards=len(patient_lab_90_now_max[((patient_lab_90_now_max['age']>=65)&\
                                      (patient_lab_90_now_max['lab_value']<'7.5'))|\
                                     ((patient_lab_90_now_max['age']<65)&\
                                      (patient_lab_90_now_max['lab_value']<'7'))])
after_bad=len(patient_lab_90_now_max[patient_lab_90_now_max['lab_value']>'9.5'])

# 输出结果
print('管理前的平均值：',patient_lab_180_30_min_mean,sep='\n',end='\n\n\n')
print('管理前的中位数：',patient_lab_180_30_min_median,sep='\n',end='\n\n\n')
print('管理后的平均值：',patient_lab_90_now_max_mean,sep='\n',end='\n\n\n')
print('管理后的中位数：',patient_lab_90_now_max_median,sep='\n',end='\n\n\n')
print('管理前后存在显著差异！',end='\n\n\n')
print('=====分割线=====')
print('管理前达标人数：',before_standards,sep='\n',end='\n\n\n')
print('管理前不良人数：',before_bad,sep='\n',end='\n\n\n')
print('管理后达标人数：',after_standards,sep='\n',end='\n\n\n')
print('管理后不良人数：',after_standards,sep='\n',end='\n\n\n')
print('管理前后存在显著差异！')

import pandas as pd 
import datetime

# 导入数据
patient=pd.read_csv(filepath_or_buffer='patient_list.csv',sep='\t')
lab=pd.read_csv(filepath_or_buffer='lab.csv',sep='\t')

# 处理数据
patient_lab=pd.merge(left=patient,right=lab,how='left',on='pat_id')
patient_lab['ago_180']=pd.to_datetime(patient_lab['create_time']).\
map(lambda x:x+datetime.timedelta(days=-180))
patient_lab['later_30']=pd.to_datetime(patient_lab['create_time']).\
map(lambda x:x+datetime.timedelta(days=30))
patient_lab['now']='2021-07-01 00:00:00'
patient_lab['ago_90']=(pd.to_datetime(arg='2021-07-01')+\
                       datetime.timedelta(days=-90)).strftime('%Y-%m-%d %H:%M:%S')
patient_lab_180_30=patient_lab[(patient_lab['lab_date']>=patient_lab['ago_180'])&\
                               (patient_lab['lab_date']<=patient_lab['later_30'])]
patient_lab_180_30_min=patient_lab_180_30[patient_lab_180_30['lab_date']==\
                                          patient_lab_180_30['lab_date'].min()]
patient_lab_90_now=patient_lab[(patient_lab['lab_date']>=patient_lab['ago_90'])&\
                               (patient_lab['lab_date']<=patient_lab['now'])]
patient_lab_90_now_max=patient_lab_90_now[patient_lab_90_now['lab_date']==\
                                          patient_lab_90_now['lab_date'].max()]
patient_lab_90_now_max=patient_lab_90_now_max[patient_lab_90_now_max\
                                              ['labitems_name']=='HbA1c']
patient_lab_180_30_min_mean=patient_lab_180_30_min['lab_value'].mean()
patient_lab_180_30_min_median=patient_lab_180_30_min['lab_value'].median()
patient_lab_90_now_max_mean=patient_lab_90_now_max['lab_value'].mean()
patient_lab_90_now_max_median=patient_lab_90_now_max['lab_value'].median()
before_standards=len(patient_lab_180_30_min[((patient_lab_180_30_min['age']>=65)&\
                                      (patient_lab_180_30_min['lab_value']<'7.5'))|\
                                     ((patient_lab_180_30_min['age']<65)&\
                                      (patient_lab_180_30_min['lab_value']<'7'))])
before_bad=len(patient_lab_180_30_min[patient_lab_180_30_min['lab_value']>'9.5'])
after_standards=len(patient_lab_90_now_max[((patient_lab_90_now_max['age']>=65)&\
                                      (patient_lab_90_now_max['lab_value']<'7.5'))|\
                                     ((patient_lab_90_now_max['age']<65)&\
                                      (patient_lab_90_now_max['lab_value']<'7'))])
after_bad=len(patient_lab_90_now_max[patient_lab_90_now_max['lab_value']>'9.5'])

# 输出结果
print('管理前的平均值：',patient_lab_180_30_min_mean,sep='\n',end='\n\n\n')
print('管理前的中位数：',patient_lab_180_30_min_median,sep='\n',end='\n\n\n')
print('管理后的平均值：',patient_lab_90_now_max_mean,sep='\n',end='\n\n\n')
print('管理后的中位数：',patient_lab_90_now_max_median,sep='\n',end='\n\n\n')
print('管理前后存在显著差异！',end='\n\n\n')
print('=====分割线=====')
print('管理前达标人数：',before_standards,sep='\n',end='\n\n\n')
print('管理前不良人数：',before_bad,sep='\n',end='\n\n\n')
print('管理后达标人数：',after_standards,sep='\n',end='\n\n\n')
print('管理后不良人数：',after_bad,sep='\n',end='\n\n\n')
print('管理前后不存在显著差异！')

import pandas as pd 
import datetime

# 导入数据
patient=pd.read_csv(filepath_or_buffer='patient_list.csv',sep='\t')
lab=pd.read_csv(filepath_or_buffer='lab.csv',sep='\t')

# 处理数据
patient_lab=pd.merge(left=patient,right=lab,how='left',on='pat_id')
doc_id=pd.DataFrame(data=patient_lab['doc_id'].drop_duplicates())
doc_id_patient_lab=pd.merge(left=doc_id,right=patient_lab,how='left',on='doc_id')
doc_id_patient_lab['ago_180']=pd.to_datetime(doc_id_patient_lab['create_time']).\
map(lambda x:x+datetime.timedelta(days=-180))
doc_id_patient_lab['later_30']=pd.to_datetime(doc_id_patient_lab['create_time']).\
map(lambda x:x+datetime.timedelta(days=30))
doc_id_patient_lab['now']='2021-07-01 00:00:00'
doc_id_patient_lab['ago_90']=(pd.to_datetime(arg='2021-07-01')+\
                              datetime.timedelta(days=-90)).\
strftime('%Y-%m-%d %H:%M:%S')
doc_id_patient_lab_180_30=doc_id_patient_lab[(doc_id_patient_lab['lab_date']>=\
                                              doc_id_patient_lab['ago_180'])&\
                                             (doc_id_patient_lab['lab_date']<=\
                                              doc_id_patient_lab['later_30'])]
doc_id_patient_lab_180_30_min=doc_id_patient_lab_180_30[doc_id_patient_lab_180_30\
                                                        ['lab_date']==\
                                                        doc_id_patient_lab_180_30\
                                                        ['lab_date'].min()]
doc_id_patient_lab_90_now=doc_id_patient_lab[(doc_id_patient_lab['lab_date']>=\
                                              doc_id_patient_lab['ago_90'])&\
                                             (doc_id_patient_lab['lab_date']<=\
                                              doc_id_patient_lab['now'])]
doc_id_patient_lab_90_now_max=doc_id_patient_lab_90_now[doc_id_patient_lab_90_now\
                                                        ['lab_date']==\
                                                        doc_id_patient_lab_90_now\
                                                        ['lab_date'].max()]
doc_id_patient_lab_90_now_max=doc_id_patient_lab_90_now_max\
[doc_id_patient_lab_90_now_max['labitems_name']=='HbA1c']
doc_id_patient_lab_180_30_min_mean=doc_id_patient_lab_180_30_min['lab_value'].mean()
doc_id_patient_lab_180_30_min_median=doc_id_patient_lab_180_30_min['lab_value'].\
median()
doc_id_patient_lab_90_now_max_mean=doc_id_patient_lab_90_now_max['lab_value'].mean()
doc_id_patient_lab_90_now_max_median=doc_id_patient_lab_90_now_max['lab_value'].\
median()
before_standards=len(doc_id_patient_lab_180_30_min[((doc_id_patient_lab_180_30_min\
                                                     ['age']>=65)&\
                                                    (doc_id_patient_lab_180_30_min\
                                                     ['lab_value']<'7.5'))|\
                                                   ((doc_id_patient_lab_180_30_min\
                                                     ['age']<65)&\
                                                    (doc_id_patient_lab_180_30_min\
                                                     ['lab_value']<'7'))])
before_bad=len(doc_id_patient_lab_180_30_min[doc_id_patient_lab_180_30_min\
                                             ['lab_value']>'9.5'])
after_standards=len(doc_id_patient_lab_90_now_max[((doc_id_patient_lab_90_now_max\
                                                    ['age']>=65)&\
                                                   (doc_id_patient_lab_90_now_max\
                                                    ['lab_value']<'7.5'))|\
                                                  ((doc_id_patient_lab_90_now_max\
                                                    ['age']<65)&\
                                                   (doc_id_patient_lab_90_now_max\
                                                    ['lab_value']<'7'))])
after_bad=len(doc_id_patient_lab_90_now_max[doc_id_patient_lab_90_now_max\
                                            ['lab_value']>'9.5'])

# 输出结果
print('管理前的平均值：',doc_id_patient_lab_180_30_min_mean,sep='\n',end='\n\n\n')
print('管理前的中位数：',doc_id_patient_lab_180_30_min_median,sep='\n',end='\n\n\n')
print('管理后的平均值：',doc_id_patient_lab_90_now_max_mean,sep='\n',end='\n\n\n')
print('管理后的中位数：',doc_id_patient_lab_90_now_max_median,sep='\n',end='\n\n\n')
print('管理前后存在显著差异！',end='\n\n\n')
print('=====分割线=====')
print('管理前达标人数：',before_standards,sep='\n',end='\n\n\n')
print('管理前不良人数：',before_bad,sep='\n',end='\n\n\n')
print('管理后达标人数：',after_standards,sep='\n',end='\n\n\n')
print('管理后不良人数：',after_bad,sep='\n',end='\n\n\n')
print('管理前后不存在显著差异！')



## 第二组（简答）
### 仅从本案例提供的信息中来看，近期化验的糖化血红蛋白是否达标，可能与什么因素相关？请简述你所考虑的用户特征、所适用的分析方法，以及工作思路和步骤。
'''不达标，可能与管理期间患者的饮食有关，饮食调控是很好的调节血糖以及糖化血红蛋白的方法；
可能与管理期间患者是否营养不良有关；也可能与管理期间患者是否患有慢性失血之类的疾病有关。。。
可以在本案例中加入可能会影响糖化血红蛋白指标的因素构建机器学习模型，通过机器学习模型来对管理
期间前后是否打标做出分类，通过特征重要性来输出其中影响最大的特征，进而指导患者在管理期间应当
着重注意哪些关键特征，使糖化血红蛋白达标改善患者病况。'''
