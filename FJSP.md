
## (一)Problem Description
 
### ● Topic:
- Processing Order
<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_OS.JPG width="650">

- Processing Time

<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_PT.JPG width="450">

### ● Aim : 
- Minimize Completion Time 
### ● Limit:
- Every job must follow a certain order
- Each operation has a different machine to choose from.
- Each machine can only do one job at a time.
### (二)Mathematical Model

### ● Notation
<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_Notation.JPG width="950">

### ● Decision Variables
<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_DV.JPG width="850">

```python
Cmax = pulp.LpVariable('Cmax',lowBound = 0, cat='Continuous')

Ci = pulp.LpVariable.dicts("start_time",
                           ((i) for i in range(1,6)),
                           lowBound=0,
                           cat='Continuous')
#i=job,j=operation,k=machine
st = pulp.LpVariable.dicts("start_time",
                           ((i, j, k) for i in range(1,6) for j in [ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0] for k in list(Processing_time.loc[(Processing_time['Operation']==j),'machine'])),
                           lowBound=0,
                           cat='Continuous')

ct = pulp.LpVariable.dicts("completed_time",
                           ((i, j, k) for i in range(1,6) for j in [ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0] for k in list(Processing_time.loc[(Processing_time['Operation']==j),'machine'])),
                           lowBound=0,
                           cat='Continuous')

x = pulp.LpVariable.dicts("X_binary_var",
                          ((i, j, k) for i in range(1,6) for j in [ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0] for k in list(Processing_time.loc[(Processing_time['Operation']==j),'machine'])),
                          lowBound=0,
                          cat='Binary')

y = pulp.LpVariable.dicts("Y_binary_var",
                          ((i, j, l, m,k) for i in range(1,6) for j in [ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0]  for l in range(1,6) for m in [ops[l-1][a] for a in range(1,5) if ops[l-1][a]!=0]  for k in list(set(list(Processing_time.loc[(Processing_time['Operation']==j),'machine']))&set(list(Processing_time.loc[(Processing_time['Operation']==m),'machine']))) if i < l ),
                          lowBound=0,
                          cat='Binary')
```

### ● Mathematical
- Target Type :
Min Cmax
```python
model = pulp.LpProblem("MIN_makespan", pulp.LpMinimize)
model += Cmax
```
- Restricted:
<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_C1.JPG width="250">

- Ensure that Operation is only executed on one machine
<img src=https://github.com/wurmen/Genetic-Algorithm-for-Job-Shop-Scheduling-and-NSGA-II/blob/master/implementation%20with%20python/GA-jobshop/picture/4.png width="350">

```python
#Add Restrictions
###Restricted###
for i in range(1,6):
    for j in [ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0]:
        ma_list=list(Processing_time.loc[(Processing_time['Operation']==j),'machine'])
        model += pulp.lpSum(x[i,j,ma] for ma in ma_list)==1
```

<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_C2.JPG width="400">

<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_C3.JPG width="500">

- If the jth operation of the i-th job is executed on Machine k, then its Operation end time is greater than or equal to the start time + job time, the operation is not executed, and the start and end times are equal to 0.

```python
for i in range(1,6):
    for j in [ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0]:
        ma_list=list(Processing_time.loc[(Processing_time['Operation']==j),'machine'])
        for k in ma_list:
            ptime=int(Processing_time.loc[(Processing_time['Operation']==j)&(Processing_time['machine']==k),['J%d'%i]].iloc[0])
            I = i
            J = j
            K = k
            model += st[I,J,K] + ct[I,J,K] <=x[I,J,K]*1000
            model += ct[I,J,K] >= st[I,J,K] +ptime-(1-x[I,J,K])*1000
```

<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_C4.JPG width="500">

<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_C5.JPG width="500">

- If two operations are to use the same machine, make sure they don't execute at the same time.

```python
for i in range(1,6):
    for l in range(1,6):
        if i<l:
            for o1 in [ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0]:
                ma1_list=list(Processing_time.loc[(Processing_time['Operation']==o1)&(Processing_time.loc[:]['J%d'%i]!=0),'machine'])
                for o2 in [ops[l-1][o] for o in range(1,5) if ops[l-1][o]!=0]:
                    ma2_list=list(Processing_time.loc[(Processing_time['Operation']==o2)&(Processing_time.loc[:]['J%d'%l]!=0),'machine'])
                    mach_set=list(set(ma1_list)&set(ma2_list))
                    if len(mach_set)>0:
                        for k in mach_set:
                            model += st[i,o1,k]>=ct[l,o2,k]-y[i,o1,l,o2,k]*1000
                            model += st[l,o2,k]>=ct[i,o1,k]-(1-y[i,o1,l,o2,k])*1000
```

<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_C6.JPG width="300">

- Ensure the order of each Job's Operation, be sure to complete the previous one before proceeding to the next Operation.

```python
for i in range(1,6):
    op_num=[ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0]
    print(op_num)
    for j in range(1,len(op_num)):
        print(i,j)
        preop_num=op_num[j-1]
        curop_num=op_num[j]
        print('pc')
        print(preop_num,curop_num)
        prema_list=list(Processing_time.loc[(Processing_time['Operation']==preop_num)&(Processing_time.loc[:]['J%d'%i]!=0),'machine'])
        curma_list=list(Processing_time.loc[(Processing_time['Operation']==curop_num)&(Processing_time.loc[:]['J%d'%i]!=0),'machine'])
        model += pulp.lpSum(st[i,curop_num,ma1] for ma1 in curma_list) >= pulp.lpSum(ct[i,preop_num,ma2] for ma2 in prema_list)
```

<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_C7.JPG width="200">

- Make sure that the final completion time of each job is the time after the last Operation of each job is completed.

```python
for i in range(1,6):
    op_list=[ops[i-1][o] for o in range(1,5) if ops[i-1][o]!=0]
    last_op=op_list[-1]
    model +=Ci[i]>=pulp.lpSum(ct[i,last_op,k] for k in list(Processing_time.loc[(Processing_time['Operation']==last_op),'machine']))
```

<img src=https://github.com/KevinLu43/JSP-by-using-Mathematical-Programming-in-Python/blob/master/Picture/FJSP_C8.JPG width="200">

- Define Makespan

```python
    model += Cmax >= Ci[i]
```       

## Solve
```python
model.solve()
pulp.LpStatus[model.status]     
```
# Print out the solution and target value
```python
for var in ct:
    var_value = ct[var].varValue
    print( var[0],var[1],var[2] ,var_value)

total_cost = pulp.value(model.objective)
print ('min cost:',total_cost)
```
-min cost: 193.0
### ● Gantt Chart
<img src=https://github.com/KevinLu43/Job-Shop-Scheduling-with-Python/blob/master/Picture/FJSP_G.JPG width="700">

### ● Reference
[Cemal Özgüven, Lale Özbakır, Yasemin Yavuz Mathematical models for job-shop scheduling problems with routing and process plan flexibility](https://www.sciencedirect.com/science/article/pii/S0307904X09002819)
