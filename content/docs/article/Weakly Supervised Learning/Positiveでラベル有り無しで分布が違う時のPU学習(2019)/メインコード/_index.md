---
title: "メインコード"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[こちら](https://github.com/MasaKat0/PUlearning/tree/master/BiasedPUlearning/PUSB)のソースコード。

# import

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.linear_model import LogisticRegression

from dataset_linear import make_data
from pusb_linear_kernel import PU
from densratio import densratio
```

色々import

# experiment

```python
def experiment(datatype, udata):
	# 
    priors = [0.2, 0.4, 0.6, 0.8]
    ite = 100
    pdata = 400
    num_basis = 300

    seed = 2018

    est_error_pu = np.zeros((len(udata), len(priors), ite))
    est_error_pubp = np.zeros((len(udata), len(priors), ite))
    est_error_dr = np.zeros((len(udata), len(priors), ite))

	# 
    for i in range(len(udata)):
        u = udata[i]
        for j in range(len(priors)):
            pi = priors[j]
            for k in range(ite):
                np.random.seed(seed)
                #PN classification
                x, t = make_data(datatype=datatype)
                x = x/np.max(x, axis=0)
                one = np.ones((len(x),1))
                x_pn = np.concatenate([x, one], axis=1)
                classifier = LogisticRegression(C=0.01, penalty='l2')
                classifier.fit(x_pn, t) 
                                
                perm = np.random.permutation(len(x))
                x_train = x[perm[:-3000]]
                t_train = t[perm[:-3000]]
                x_test = x[perm[-3000:]]
                t_test = t[perm[-3000:]]
                
                xp = x_train[t_train==1]
                one = np.ones((len(xp),1))
                xp_temp = np.concatenate([xp, one], axis=1)
                xp_prob = classifier.predict_proba(xp_temp)[:,1]
                #xp_prob /= np.mean(xp_prob)
                xp_prob = xp_prob**20
                xp_prob /= np.max(xp_prob)
                rand = np.random.uniform(size=len(xp))
                temp = xp[xp_prob > rand]
                while (len(temp) < pdata):
                    rand = np.random.uniform(size=len(xp))
                    temp = np.concatenate([temp, xp[xp_prob > rand]], axis=0)
                xp = temp
                perm = np.random.permutation(len(xp))
                xp = xp[perm[:pdata]]
                updata = np.int(u*pi)
                undata = u - updata
                
                xp_temp = x_train[t_train==1]
                xn_temp = x_train[t_train==0]
                perm = np.random.permutation(len(xp_temp))
                xp_temp = xp_temp[perm[:updata]]
                
                perm = np.random.permutation(len(xn_temp))
                xn_temp = xn_temp[perm[:undata]]
                xu = np.concatenate([xp_temp, xn_temp], axis=0)
                
                x = np.concatenate([xp, xu], axis=0)

                tp = np.ones(len(xp))
                tu = np.zeros(len(xu))
                t = np.concatenate([tp, tu], axis=0)
                
                updata = np.int(1000*pi)
                undata = 1000 - updata
                
                xp_test = x_test[t_test == 1]
                perm = np.random.permutation(len(xp_test))
                xp_test = xp_test[perm[:updata]]
                xn_test = x_test[t_test == 0]
                perm = np.random.permutation(len(xn_test))
                xn_test = xn_test[perm[:undata]]
                
                x_test = np.concatenate([xp_test, xn_test], axis=0)
                tp = np.ones(len(xp_test))
                tu = np.zeros(len(xn_test))
                t_test = np.concatenate([tp, tu], axis=0)
                
                pu = PU(pi=pi)
                x_train = x
                res, x_test_kernel = pu.optimize(x, t, x_test)
                acc1 = pu.test(x_test_kernel, res, t_test, quant=False)
                acc2 = pu.test(x_test_kernel, res, t_test, quant=True, pi=pi)
                
                result = densratio(x_train[t==1], x_train[t==0])
                r = result.compute_density_ratio(x_test)
                temp = np.copy(r)
                temp = np.sort(temp)
                theta = temp[np.int(np.floor(len(x_test)*(1-pi)))]
                pred = np.zeros(len(x_test))
                pred[r > theta] = 1
                acc3 = np.mean(pred == t_test)
                
                est_error_pu[i, j, k] = acc1
                est_error_pubp[i, j, k] = acc2
                est_error_dr[i, j, k] = acc3

                seed += 1
                
                print(acc1)
                print(acc2)
                print(acc3)
    
    est_error_pu_mean = np.mean(est_error_pu, axis=2)
    est_error_pubp_mean = np.mean(est_error_pubp, axis=2)
    est_error_dr_mean = np.mean(est_error_dr, axis=2)
    est_error_pu_std = np.std(est_error_pu, axis=2)
    est_error_pubp_std = np.std(est_error_pubp, axis=2)
    est_error_dr_std = np.std(est_error_dr, axis=2)
    return est_error_pu_mean, est_error_pubp_mean, est_error_pu_std, est_error_pubp_std, est_error_dr_mean, est_error_dr_std

```