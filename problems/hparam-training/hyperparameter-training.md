


Hyperparameter Training



RESOURCES
- Job Scheduler is a bit of a similar problem






PROMPT
```
How would you go about designing the infrastructure to support running trains with different hparams combinations and returning the highest-score hparam combination in an engineering organization?

For the purpose of this problem, we can consider “training a model” as a black box:

Inputs:
A set of hyperparameter candidate values to be tested
Model specification
Training + validation data

Output:
An F1 score, represented as a float from 0-1
The "best" hyperparam values that resulted in that F1 score

What are some choices for components of the system? How do they work together? What trade-offs could you consider? What failure modes are and aren't handled?

You can assume the following:

Training a single model takes multiple hours on a single machine
Machines running training jobs requires special hardware like a GPU
There is a quota on the maximum number of GPUs you can provision at one time
Training and Validation data is the order of 10s of GBs
You have access to a cloud provider like AWS or GCP
There is a way to monitor the progress and performance of a training run.

Inputs to the system:
List of Hyperparameters, e.g. [a, b], and their candidate values e.g. [(a=1,b=x), (a=1, b=y), …]
You're passed all combinations that we want tested
(i.e. no need to explode these into all possible values)
Specification of model
training+validation data

Expected Output:
List of hyperparameter tuples, the associated score
Sorted descending by score
This can be made available in any way that makes sense, through a dashboard, as a file, in the output of a job, etc.

So the expected flow is a User tries to interact with the system by providing the inputs (list of hyper-parameters, specification of model, training+validation data)

Can someone come up with a scalable system design for this?
```







REQUIREMENTS
- You're passed all combinations that we want tested
- Associated F1 score for a set of hyperparameters
- Training a single model takes multiple hours on a single machine
- Machines running training jobs requires special hardware like a GPU
- There is a quota on the maximum number of GPUs you can provision at one time










CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:





https://scikit-learn.org/stable/modules/grid_search.html

https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html






- search autocompletes of amazon.com's search bar
- item purchases (similar item recommendation)










Can we use Kubeflow for running these experiments ?







Looking at the schema for metadata store, wondering do we need nosql solution or can we use relation DB itself here ? We might get this by doing the back off the envleope calculation. Thoughts ?
\---
relational DOES actually work here just fine :)







































