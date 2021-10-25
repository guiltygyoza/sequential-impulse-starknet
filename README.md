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
3. Identify all "position constraints". Iterate through the contraints and resolve each one sequentially, where the resolution of any constraint returns a change in the velocities & angular velocities of the bodies involved in the constraint, which is applied and used by the resolution of the next constraint.
4. Use the finalized next velocities `(V_Cx_nxt, V_Cy_nxt)` and angular velocities `ω_nxt` to obtain `(Cx_nxt, Cy_nxt)` and `θ_nxt` from `(Cx',Cy')` and `θ'` by Euler method.

A constraint can be:
- contact constraint, which when stated from the velocity standpoint requires `the projection of the relative velocity of the two contact points onto the contact normal is zero`
- friction constraint, which when stated from the velocity standpoint requires `the projection of the relative velocity of the two contact points onto the contact tangent plane is zero`
- a joint that restricts degree(s) of freedom
- etc

This prototype focuses on resolving contact constraint, taking restitution into consideration. To formalize, given a contact position constraint:
```
C: (PB-PA) • n ≥ 0
```
where `PB` and `PA` are contact points and `n` is contact normal, the corresponding velocity constraint `dC/dt` can be expressed in the following form:
```
dC/dt: JV + b = 0
```
where V is the velocities and angular velocities bundled together as `V = [Vax, Vay, Vaz, ωax, ωay, ωaz, Vbx, Vby, Vbz, ωbx, ωby, ωbz]`, `b=0` (we will modify this later), and `J` is a 12x1 row-vector Jacobian.

After simplification, `J` looks like this (in three dimension):
```
J = [-nx, -ny, 0, 0, 0, (-rax*ny+ray*nx), nx, ny, 0, 0, 0, (rbx*ny-rby*nx)]
```
where `ra=(rax,ray)` is the vector from A's center of mass to A's contact point. likewise for `rb`.

The change in `V` can be expressed as:

`delta V = M^-1 J^T λ`, where (`M = mass * I_3x3` while `I` is inertia tensor; `λ` is the Lagrangian Multiplier):

```
       [ Ma 0  0  0 ]
M^-1 = [ 0  Ia 0  0 ]
       [ 0  0  Mb 0 ]
       [ 0  0  0  Ib]
       
       
λ = -(JV + b) / (J M^-1 J^T)
```

Last but not least, to stabilize the bodies at collision as well as adding restitution, instead of `b = 0`, we have:

```
b = b_B + b_R
```
where `b_B` is the Baumgarte Stabilization term:
```
b_B = -β * d / dt (d is penetration depth. 0<β<1 and is usually chosen close to 0)
```
and `b_R` is the restitution term:
```
b_R = C_R * 'closing velocity vector of two contact points' dot n
(C_R is the coefficient of restitution; n is the contact normal vector)
```

Note:
- tunneling effect is not dealt with yet.

Reference:
- http://allenchou.net/2013/12/game-physics-constraints-sequential-impulse/
- http://allenchou.net/2013/12/game-physics-resolution-contact-constraints/

