# US-Election-Commision-Dataset-Analysis

import pandas as pd
import numpy as np

df = pd.read_csv('P00000001-ALL.csv')

unique = df['cand_nm'].unique()
parties = {'Bachmann, Michelle': 'Republican',
 'Cain, Herman': 'Republican',
 'Gingrich, Newt': 'Republican',
 'Huntsman, Jon': 'Republican',
 'Johnson, Gary Earl': 'Republican',
 'McCotter, Thaddeus G': 'Republican',
 'Obama, Barack': 'Democrat',
 'Paul, Ron': 'Republican',
 'Pawlenty, Timothy': 'Republican',
 'Perry, Rick': 'Republican',
 "Roemer, Charles E. 'Buddy' III": 'Republican','Romney, Mitt': 'Republican',
 'Santorum, Rick': 'Republican'}
df.cand_nm[123456:123461].map(parties)
df['party'] = df.cand_nm.map(parties)
df['party'].value_counts()
df = df[df['contb_receipt_amt'] > 0]
df.contbr_occupation.value_counts()[:10]

occ_mapping = {
 'INFORMATION REQUESTED PER BEST EFFORTS' : 'NOT PROVIDED',
 'INFORMATION REQUESTED' : 'NOT PROVIDED',
 'INFORMATION REQUESTED (BEST EFFORTS)' : 'NOT PROVIDED',
 'C.E.O.': 'CEO'
}
f = lambda x: occ_mapping.get(x,x)
df.contbr_occupation = df.contbr_occupation.map(f)

emp_mapping = {
 'INFORMATION REQUESTED PER BEST EFFORTS' : 'NOT PROVIDED',
 'INFORMATION REQUESTED' : 'NOT PROVIDED',
 'SELF' : 'SELF-EMPLOYED',
 'SELF EMPLOYED' : 'SELF-EMPLOYED',
}

f1 = lambda x: emp_mapping.get(x,x)
df.contbr_employer = df.contbr_employer.map(f1)
by_occupation = df.pivot_table('contb_receipt_amt',index = 'contbr_occupation',columns='party', aggfunc = 'sum')
by_occupation
over_2mm = by_occupation[by_occupation.sum(1) > 2000000]
over_2mm.plot(kind='barh')




df_mrbo = df[df['cand_nm'].isin(['Obama, Barack','Romney, Mitt'])]
def funct(group,key,n=5):
    totals = group.groupby(key)['contb_receipt_amt'].sum()
    return totals.order(ascending=False)[-n:]
grouped = df_mrbo.groupby('cand_nm')

#grouped.apply(funct, 'contbr_occupation',n=7)

bins = np.array([0,1,10,100,1000,10000,100000,1000000,10000000])
labels = pd.cut(df_mrbo.contb_receipt_amt,bins)
grouped = df_mrbo.groupby(['cand_nm',labels])

grouped.size().unstack(0)
bucket = grouped.contb_receipt_amt.sum().unstack(0)
normed_sum = bucket.div(bucket.sum(axis = 1),axis=0)
normed_sum.plot(kind='barh',stacked=True)
