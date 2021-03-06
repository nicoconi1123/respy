.. _randomness-and-reproducibility:

Randomness and Reproducibility
==============================

**respy** embraces randomness to study individual behavior under risk. At the same time,
it is crucial to make results reproducible. To build a reproducible model, users must
define three seeds for the solution, simulation and estimation of the model in the
options. This allows to study the impact of randomness for each of the components
independently.

.. code-block:: python

    options = {"solution_seed": 1, "simulation_seed": 2, "estimation_seed": 3}

The seeds for the solution, simulation and estimation are used to draw a 3-, 5- and
7-digit seed sequence [#f1]_. The first 100 seeds in the sequences are reserved for
randomness in the startup of functions like :func:`~respy.simulate.simulate` or
:func:`~respy.likelihood.log_like`, e.g., to create draws from a uniform distribution.
All other seeds are used during the iterations of those functions and reset to the
initial value at the begin of every iteration.

As a general rule, models in **respy** are reproducible or use the same randomness as
long as only model parameters are changed, for example utility parameters, but the
structure of the model stays the same. The following list includes example of structural
changes to the model.

- Changing the choice set (forms of renaming, removing choices).
- Changing the initial conditions (experiences, lagged choices, type probabilities).
- Changing the Monte Carlo integrations (sequence, number of draws).
- Using interpolation and changing the number of non-interpolated states.
- Removing states from the state space via filters.

In the following, we document for each module the functions which use seeds to control
randomness.


respy.shared
------------

.. currentmodule:: respy.shared

The function :func:`create_base_draws` is used in all parts, solution, simulation, and
estimation, to generate random draws. :func:`transform_base_draws_with_cholesky_factor`
transforms the base draws to the variance-covariance matrix implied by the model
parameters.

.. autosummary::

    create_base_draws
    transform_base_draws_with_cholesky_factor


respy.solve
-----------

Routines under ``respy.solve`` use a seed from the sequence initialized by
``options["solution_seed"]`` to control randomness. Apart from the draws,
:func:`~respy.solve.solve` relies on the following function.

.. currentmodule:: respy.interpolate

.. autosummary::

    _get_not_interpolated_indicator


respy.simulate
--------------

Routines under ``respy.simulate`` use a seed from the sequence of
``options["simulation_seed"]`` to control randomness. Apart from the draws,
:func:`~respy.simulate.simulate` relies on the following function to generate
starting values for simulated individuals (experiences, types, etc.).

.. currentmodule:: respy.simulate

.. autosummary::

    _sample_characteristic


respy.likelihood
----------------

Routines under ``respy.likelihood`` use a seed from the sequence specified under
``options["estimation_seed"]`` to control randomness. The seed is used to create the
draws to simulate the probability of observed wages with
:func:`~respy.shared.create_base_draws`.


respy.tests.random_model
------------------------

The regression tests are run on truncated data set which contains truncated history of
individuals or missing wage information. The truncation process is controlled via a seed
in the sequence initialized by ``options["simulation_seed"]``.

.. currentmodule:: respy.tests.random_model

.. autosummary::

    simulate_truncated_data


.. seealso::

    See `Random number generator seed mistakes <https://www.johndcook.com/blog/2016/01/
    29/random-number-generator-seed-mistakes/>`_ for a general introduction to seeding
    problems.

    See `this comment <https://www.johndcook.com/blog/2016/01/29/
    random-number-generator-seed-mistakes/#comment-704037>`_ in the same post which
    verifies independence between sequential seeds.

    NumPy documentation on their `RandomState object <https://docs.scipy.org/doc/
    numpy-1.15.0/reference/generated/numpy.random.RandomState.html>`_ which wraps the
    pseudo-random number generator `Mersenne Twister <https://docs.scipy.org/doc/numpy/
    reference/random/bit_generators/mt19937.html>`_.


.. rubric:: Footnotes

.. [#f1] The need for seed sequences became apparent in `#268
         <https://github.com/OpenSourceEconomics/respy/pull/268>`_.
