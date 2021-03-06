.. _when:

When
----

.. index:: ! when

By using ``when``, we can express conditions we are interested in
reacting to and what we wish to do in response to them.  In this
section, we'll review the key ideas behind ``when`` statements.  A
``when`` statement has the following general form:

.. code-block:: modelica

    when expr then
      // Statements
    end when;

.. _if-vs-when:

``if`` vs. ``when``
^^^^^^^^^^^^^^^^^^^

In our discussion on :ref:`hysteresis`, we briefly discussed the
difference between an ``if`` statement and a ``when`` statement.  The
statements in a ``when`` statement become active only for an instant
when the triggering conditional expression becomes true.  At all other
times, the ``when`` statement has no effect.  An ``if`` statement or
``if`` expression remains active as long as the conditional expression
is true.  If the ``if`` statement or ``if`` expression includes an
``else`` clause then some branch will always be active.

Expressions
^^^^^^^^^^^

Most of the time, the expression ``expr`` is going to be a conditional
expression and usually it will involve relational operators.  Typical
examples of expressions frequently used with a ``when`` statement
would be ``time>=2.0``, ``x>=y+2``, ``phi<=prev_phi`` and so on.
Recall from our discussion of the :ref:`interval-measurement` speed
estimation algorithm that you should **almost always** put the ``pre``
operator around any variables in ``expr`` that also appear inside the
``when`` statement.

In the :ref:`bouncing-ball` example, we saw a case where ``expr`` was
not a (scalar) conditional expression, but rather a vector of
conditional expressions.  Recall from that discussion that the
``when`` statement becomes active if **any** of the conditions in the
vector of expressions becomes true.

Statements
^^^^^^^^^^

A ``when`` statement is used to define new values for some variables.
These new values can be assigned in one of two ways.  The first is by
introducing an equation of the form:

.. code-block:: modelica

  var = expr;

In this case, the variable ``var`` will be given the value of
``expr``.  Within ``expr``, the ``pre`` operator should be used when
referring to the pre-event value for a variable.  Any variable
assigned in this way is a discrete variable.  This means that its
value only changes during events.  In other words, it will be a
piecewise constant function.  Note, a variable assigned in this way
cannot be continuous over any interval in the simulation.

If you want to explicitly mark a variable as discrete, you can prefix
it with the ``discrete`` qualifier (as we saw in the
:ref:`sample-and-hold` example earlier in this chapter) although this
is not strictly necessary.  By adding the ``discrete`` qualifier you
ensure that the variable's value must be determined within a ``when``
statement.

.. index:: reinit

The other way a variable can be given a value within a ``when``
statement, as demonstrated in the :ref:`bouncing-ball` example, is by
using the ``reinit`` operator.  In that case, the statement within the
``when`` statement will take the form:

.. code-block:: modelica

    reinit(var, expr);

When using the ``reinit`` operator, the variable, ``var``, **must be a
state**.  In other words, its solution must arise from solving a
differential equation.  The use of ``reinit`` on such a variable has
the effect of stopping the integration process, changing the value of
the state (and any other states that have the ``reinit`` operator
applied to them within the same ``when`` statement) and then resuming
integration using what are effectively a new set of initial
conditions.  The values of all other states not re-initialized with the
``reinit`` operator remain unchanged.

.. _algorithm-sections:

``algorithm`` Sections
^^^^^^^^^^^^^^^^^^^^^^

One final note about ``when`` statements is how they interact with the
"single assignment" rule in Modelica.  This rule, from the
specification, states that there must be exactly one equation used to
determine the value of each variable.  As we saw in the sections on
:ref:`speed-measurement` and :ref:`hysteresis`, it is sometimes
necessary (or least clearer) to express behavior in terms of multiple
assignments.  In those cases, if all the assignments are included
within a single ``algorithm`` section, they are effectively combined
into a single equation.  However, doing so will limit the compiler's
ability to perform symbolic manipulation and, therefore, may impact
simulation performance and/or reusability of the models.

It is also worth noting that if the semantics of an ``algorithm``
section are needed during initialization, Modelica includes an
``initial algorithm`` section that is analogous to the ``initial
equation`` discussed in the previous section on :ref:`initialization`.
The ``initial algorithm`` section will be applied only during the
initialization phase to determine initial conditions, just like an
``initial equation`` section, but the ``initial algorithm`` section
will allow multiple assignments to the same variable.  The same
caveats apply with respect to symbolic manipulation.
.. todo:: what would be the purpose in having multiple assignments to the same variable?is the order important like in non-initial algorithm sections where
the last assigment is the one used?
for any algorithm section (initial or otherwise):
are the statements executed sequentially so that
x = 2; y = x; z = y is well defined?

