## Proof of correctness for EDT

Note that I'm not entirely confident that the implementation actually does what
this "proof of correctness" says it should. If so, that's the implementation's
fault.

Using notation:

- `E[]` be expected value
- `P[]` be probability
- `Sum[x in X] Y` mean the sum over `x` in the set `X` of `Y`
- `x y` means `x * y` if `x` and `y` are probabilities, or `x and y` if they're
  probabilistic events.

Let:

- `U` be utility according to the agent
- `e0` be the starting event
- `D` be the set of events at which the decision occurs
- `A` be the set of events immediately following the action being taken
- `F` be the set of final events
- `B` be the set of behavior that agents could have (a mapping from agent and
  decision to action)

EDT is supposed to pick the best action according to:

    E[U | e0 D A]

Now we can do some algebra:

    E[U | e0 D A]
    = Sum[e2 in F] U(e2) P[e2 | e0 D A]

    P[e2 | e0 D A]
    = P[D A e2 | e0] / P[D A | e0]
    = Sum[e1 in D] Sum[e1' in D A] P[e1 e1' e2 | e0]
    / Sum[e1 in D] Sum[e1' in D A] P[e1 e1' | e0]

    --denominator
    P[e1 e1' | e0] = Sum[b in B] P[e1 e1' b | e0]
                                 ----------------
                                  (*)

    -- numerator
    P[e1 e1' e2 | e0]
    = Sum[b in B] P[e1 e1' e2 b | e0]
    = Sum[b in B] P[e2 | e0 e1 e1' b] P[e1 e1' b | e0]
    = Sum[b in B] P[e2 | e1' b]       P[e1 e1' b | e0]
                  ---------------     ---------------
                  Sim(e1', e2, b)      (*)

    -- (*)
    P[e1 e1' b | e0]
    = P[e1' | e0 e1 b]  P[e1 b | e0]
    = P[e1' | e0 e1 b]  P[e1 | e0 b]  P[b | e0]
    = P[e1' | e0 e1 b]  P[e1 | e0 b]  P[b]
       --------------   ------------  -------
       action prob      Sim(e0,e1,b)  prior

All together, we get the giant fraction:

    Sum[e2 in F] Sum[e1 in D] Sum[e1' in A D] Sum[b in B]
    U(e2) Sim(e1', e2, b) P[e1' | e0 e1 b] Sim(e0, e1, b) P[b]
    ----------------------------------------------------------
    Sum[e1 in D] Sum[e1' in A D] Sum[b in B]
    P[e1' | e0 e1 b] Sim(e0, e1, b) P[b]

While doing the algebra, we made the following assumptions:

- Utility comes only from the set `F` of final outcomes. So
  `E[U|e] = Sum[e' in F] U(e') P[e' | e]`
- Behaviors `B` are complete and disjoint. So
  `P[e] = Sum[b in B] P[b e]`
- Simulation computes `Sim(e, e', b) = P[e' | e b]`
- An agent's behavior `b in B` is independent of the starting event `e0`.
  That is to say: no one was placed in this dilemma _because of_ how they will
  act in it.
- `e0 e1 e1' b` = `e1' b` -- This needs further justification. It's saying that
  the `e1'`s that we consider (that is, the events immediately following us
  taking action A) always follow `e0` (the start of the scenario) and `e1`
  (some event in D at which we're making the decision). Which should be true in
  practice, but I'm not sure where exactly this assumption is supposed to live.
