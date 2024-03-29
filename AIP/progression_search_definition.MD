# Progression Search Formal Definition

Correction: We should use "ordering constraint" instead of "prec" in the example images

## Unconstrained Task

![image](https://github.com/hughiephan/DPL/assets/16631121/ab308075-ad15-4270-9f25-0b426b822d45)

A task id $t$ is unconstrained in a task network $(T, \prec, \alpha)$ if and only if there is no task $\acute{t} \in T$ such that $(\acute{t}, t) \in \prec)$

Explanation: Unconstrained tasks are tasks that have no predecessors in the network. So in this case there should not be any task $\acute{t}$ before our unconstrained tasks.

Example: `t1` and `t2.1` are our unconstrained tasks.
```
tn2: 
  task: t1, t2.1, t2.2, t2.3, t2.4
  ordering-constraint: <t2.1, t2.2, t2.3, t2.4>
  alpha:
    t1: deliver(P1, B),
    t2.1: get-to(T2, C),
    t2.2: pick-up(T2, C, P2),
    t2.3: get-to(T2, D),
    t2.4: drop(T2, D, P2)
```

## Search Node 
![image](https://github.com/hughiephan/DPL/assets/16631121/35562816-103f-44f2-984b-dd9f784049cd)

$$
n = (s, (T, \prec, \alpha), \pi)
$$

Search Node consists of:

- $s$: Current state
- $(T, \prec, \alpha)$: Task network
- $\pi$: Sequence of the current plan

## Fringe

![image](https://github.com/hughiephan/DPL/assets/16631121/39bd823e-2d54-4905-ae80-7149ce8c406b)

Fringe is a list of search nodes. It is like a stack, but we can control the position of search nodes in the fringe based on some cost of heuristics.

Example: 

Here's the Fringe at first.

```
fringe:
  fringe_position_0:
    state: s0
    tnI: 
      task: t1, t2
      ordering-constraint: <>
      alpha:
        t1: deliver(P1,B),
        t2: deliver(P2,D)
    pi: <>
```

Here's the Fringe after we run for a while.

```
fringe:
  ...
  fringe_position_X:
    state: s2.2.1
    tn2.2.1: 
      task: t1.1, t1.2, t1.3, t1.4, t2.1.1, t2.2, t2.3, t2.4
      ordering-constraint: 
        <t1.1, t1.2, t1.3, t1.4>,
        <t2.1.1, t2.2, t2.3, t2.4>
      alpha:
        t1.1: get-to(T1, A),
        t1.2: pick-up(T1, A, P1),
        t1.3: get-to(T1, B),
        t1.4: drop(T1, B, P1),
        t2.1.1: no-op(),
        t2.2: pick-up(T2, C, P2),
        t2.3: get-to(T2, D),
        t2.4: drop(T2, D, P2)
    pi: <>
  ...
```

Here's the Fringe after we run a bit longer, and now we have the task `no-op()` for the plan sequence.
```
fringe:
  ...
  fringe_position_Y:
    state: s2.2.1.2
    tn2.2.1: 
      task: t1.1, t1.2, t1.3, t1.4, t2.1.1, t2.2, t2.3, t2.4
      ordering-constraint: 
        <t1.1, t1.2, t1.3, t1.4>,
        <t2.2, t2.3, t2.4>
      alpha:
        t1.1: get-to(T1, A),
        t1.2: pick-up(T1, A, P1),
        t1.3: get-to(T1, B),
        t1.4: drop(T1, B, P1),
        t2.2: pick-up(T2, C, P2),
        t2.3: get-to(T2, D),
        t2.4: drop(T2, D, P2)
    pi: <no-op()>
  ...
```

## Standard Progression Search Algorithm

01. $fringe \gets { \{(s_0, tn_I, \epsilon) \} }$ (The fringe will be initialized with the initial search node)

02. While $fringe \neq \emptyset$ Do (If the fringe is not empty, continue to process)

03. &nbsp;&nbsp; $n \xleftarrow{} fringe.pop()$ (We pop the search node out to process it)

04. &nbsp;&nbsp; If $n.isgoal$ then return $n$ (If the search node's task network is the goal then we just return it as solution)

05. &nbsp;&nbsp; $U  \xleftarrow[]{} n.unconstrainedNodes$ (We get the unconstrained nodes to process)

06. &nbsp;&nbsp; For $t \in U$ do (Get the tasks from the unconstrained nodes)
 
07. &nbsp;&nbsp;&nbsp;&nbsp; If $isPrimitive(t)$ then (The tasks need to be primitive)

08. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If $isApplicable(t)$ then (The tasks need to be applicable)

09. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\acute{n} \xleftarrow[]{} n.apply(t)$

10. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $fringe.add(\acute{n})$

11. &nbsp;&nbsp;&nbsp;&nbsp; else (If the task is compound then we need to decompose it)

12. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; For $m \in t.methods$ do

13. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $\acute{n} \xleftarrow[]{} n.decompose(t,m)$

14. &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $fringe.add(\acute{n})$ (Add the already decomposed task into the fringe so we can process it)

15. Return $unsolvable$

Fringe will be initialized with this search node:
- $s_0$: initial state
- $tn_I$: initial task network
- $\epsilon$: an empty sequence

Explanation:
- To determine whether an action is applicable in a given state, the planner checks whether all the preconditions of that action are true in the current state. If all preconditions are satisfied, the action is considered applicable in the current state and can be considered as a candidate for execution. For unconstrained action, there is: No possible modification when it is not applicable, and exactly one modfiication when it is applicable. With `modification` means progressing the action and updating the state
- We can determine the heuristic cost of the search nodes in the fringe and sort them

## Progression

$$
\acute{n} = (\acute{s}, (\acute{T}, \acute{\prec}, \acute{\alpha}), \acute{\pi}
$$ 

Given:
- a search node $n = (s, T, \prec, \alpha, \pi)$
- $t \in T$: unconstrained task
- $\acute{n}$: search node from progressing $t$

With:
- $\acute{s} = \gamma ( \alpha(t), s )$ 
- $\acute{T} = T$ \ { $t$ }
- $\acute{\prec} = \prec$ \ { $(t,\acute{t}) | \acute{t} \in T$ }
- $\acute{\alpha} = \alpha$ \ { $t \mapsto \alpha(t)$ }  
- $\acute{\pi} = \pi \cdot \alpha(t)$

Explanation:
- Transition function gamma $\gamma$ to transform the task names and state to a new state.
- List of new tasks should remove the unconstrained task we just processed.
- The new precondition remove the precondition relationship between the unconstrained task 
- The $\alpha$ mapping of the unconstrained task will be removed also.
- The goal $\pi$ is also updated by adding the task names 

## Solution

![image](https://github.com/hughiephan/DPL/assets/16631121/9a0f496c-a3ce-4390-af3b-77b8984f12f8)

$$
n = (s, (T, \prec, \alpha), \pi)
$$

The node will be a solution if and only if:
- Task network is empty $T = \emptyset$
- State is goal state $s \supseteq g$

With:
- $\pi$: generated plan

Explanation:
- When we have processed all the tasks in the task network, then the task network will be empty and we are now done.
- Even though not mention but I think we need a Goal Test to determines whether a given state is a goal state.

## Solution Criteria
$$
tn = (T, \prec, \alpha)
$$

The task network $tn$ is a solution if and only if:
- $tn_I \xrightarrow{*} tn$: can be reached by decomposing the initial task network
- $\forall(t) \in T: \alpha(t) \in N_P$: all task names are primitive

## References
- [1] Höller, Daniel & Bercher, Pascal & Behnke, Gregor & Biundo, Susanne. (2020). HTN Planning as Heuristic Progression Search. Journal of Artificial Intelligence Research. 67. 835-880. 10.1613/jair.1.11282. 
- [2] https://cs.stanford.edu/people/eroberts/courses/soco/projects/2003-04/intelligent-search/intro.htm
