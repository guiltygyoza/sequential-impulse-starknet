# Sequential Impulse on StarkNet
This is a prototype of running sequential impulse for 2D physics simulation on StarkNet.

### Recap of sequential impulse
Sequential Impulse is a famous method for constraint resolution proposed and popularized by [Erin Catto](https://twitter.com/erin_catto), the creator of the physics engine Box2D which powers *Angry Bird*.

Each physics body (body in short) is described by the following set of attributes:
- center-of-mass coordinate `(Cx, Cy)`
- center-of-mass velocity `(V_Cx, V_Cy)`
- orientation `θ`
- angular velocity around its center-of-mass `ω`


The computations involved to advance the physics system at every time step are, in order:
1. For each physics body (body in short), calculate its "candidate" next center-of-mass coordinate `(Cx',Cy')` and next orientation `θ'` based on current velocity `(V_Cx, v_Cy)` and angular velocity `ω` by Euler method.
2. Apply force e.g. gravity to calculate its "candidate" next velocity `(V_Cx', V_Cy')`
3. Identify all "position constraints". Iterate through the contraints and resolve each one sequentially, where the resolution of any constraint returns a change to the velocities & angular velocities of the bodies involved in the constraint, which is applied and used by the resolution of the next constraint.
4. Use the finalized next velocities `(V_Cx_nxt, V_Cy_nxt)` and angular velocities `ω_nxt` to obtain `(Cx_nxt, Cy_nxt)` and `θ_nxt` from `(Cx',Cy')` and `θ'` by Euler method.

Note:
- tunneling effect is not dealt with yet.

Reference:
- http://allenchou.net/2013/12/game-physics-constraints-sequential-impulse/
- http://allenchou.net/2013/12/game-physics-resolution-contact-constraints/

