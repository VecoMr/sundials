.. ----------------------------------------------------------------
   Programmer(s): Daniel R. Reynolds @ SMU
   ----------------------------------------------------------------
   SUNDIALS Copyright Start
   Copyright (c) 2002-2025, Lawrence Livermore National Security
   and Southern Methodist University.
   All rights reserved.

   See the top-level LICENSE and NOTICE files for details.

   SPDX-License-Identifier: BSD-3-Clause
   SUNDIALS Copyright End
   ----------------------------------------------------------------

.. _ARKODE.Usage.ARKStep.UserCallable:

ARKStep User-callable functions
================================

This section describes the ARKStep-specific functions that may be called
by the user to setup and then solve an IVP using the ARKStep time-stepping
module.  The large majority of these routines merely wrap :ref:`underlying
ARKODE functions <ARKODE.Usage.UserCallable>`, and are now deprecated
-- each of these are clearly marked.  However, some
of these user-callable functions are specific to ARKStep, as explained
below.

As discussed in the main :ref:`ARKODE user-callable function introduction
<ARKODE.Usage.UserCallable>`, each of ARKODE's time-stepping modules
clarifies the categories of user-callable functions that it supports.
ARKStep supports *all categories*:

* temporal adaptivity
* implicit nonlinear and/or linear solvers
* non-identity mass matrices
* relaxation Runge--Kutta methods

ARKStep also has forcing function support when converted to a
:c:type:`SUNStepper` or :c:type:`MRIStepInnerStepper`. See
:c:func:`ARKodeCreateSUNStepper` and :c:func:`ARKStepCreateMRIStepInnerStepper`
for additional details.


.. _ARKODE.Usage.ARKStep.Initialization:

ARKStep initialization and deallocation functions
------------------------------------------------------


.. c:function:: void* ARKStepCreate(ARKRhsFn fe, ARKRhsFn fi, sunrealtype t0, N_Vector y0, SUNContext sunctx)

   This function creates an internal memory block for a problem to be
   solved using the ARKStep time-stepping module in ARKODE.

   **Arguments:**
      * *fe* -- the name of the C function (of type :c:func:`ARKRhsFn()`)
        defining the explicit portion of the right-hand side function in
        :math:`M(t)\, y'(t) = f^E(t,y) + f^I(t,y)`.
      * *fi* -- the name of the C function (of type :c:func:`ARKRhsFn()`)
        defining the implicit portion of the right-hand side function in
        :math:`M(t)\, y'(t) = f^E(t,y) + f^I(t,y)`.
      * *t0* -- the initial value of :math:`t`.
      * *y0* -- the initial condition vector :math:`y(t_0)`.
      * *sunctx* -- the :c:type:`SUNContext` object (see :numref:`SUNDIALS.SUNContext`)

   **Return value:**  If successful, a pointer to initialized problem memory
   of type ``void*``, to be passed to all user-facing ARKStep routines
   listed below.  If unsuccessful, a ``NULL`` pointer will be
   returned, and an error message will be printed to ``stderr``.


.. c:function:: void ARKStepFree(void** arkode_mem)

   This function frees the problem memory *arkode_mem* created by
   :c:func:`ARKStepCreate`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**  None

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeFree` instead.


.. _ARKODE.Usage.ARKStep.Tolerances:

ARKStep tolerance specification functions
------------------------------------------------------


.. c:function:: int ARKStepSStolerances(void* arkode_mem, sunrealtype reltol, sunrealtype abstol)

   This function specifies scalar relative and absolute tolerances.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *reltol* -- scalar relative tolerance.
      * *abstol* -- scalar absolute tolerance.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_NO_MALLOC*  if the ARKStep memory was not allocated by the time-stepping module
      * *ARK_ILL_INPUT* if an argument had an illegal value (e.g. a negative tolerance).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSStolerances` instead.


.. c:function:: int ARKStepSVtolerances(void* arkode_mem, sunrealtype reltol, N_Vector abstol)

   This function specifies a scalar relative tolerance and a vector
   absolute tolerance (a potentially different absolute tolerance for
   each vector component).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *reltol* -- scalar relative tolerance.
      * *abstol* -- vector containing the absolute tolerances for each
        solution component.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_NO_MALLOC*  if the ARKStep memory was not allocated by the time-stepping module
      * *ARK_ILL_INPUT* if an argument had an illegal value (e.g. a negative tolerance).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSVtolerances` instead.


.. c:function:: int ARKStepWFtolerances(void* arkode_mem, ARKEwtFn efun)

   This function specifies a user-supplied function *efun* to compute
   the error weight vector ``ewt``.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *efun* -- the name of the function (of type :c:func:`ARKEwtFn()`)
        that implements the error weight vector computation.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_NO_MALLOC*  if the ARKStep memory was not allocated by the time-stepping module

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeWFtolerances` instead.



.. c:function:: int ARKStepResStolerance(void* arkode_mem, sunrealtype rabstol)

   This function specifies a scalar absolute residual tolerance.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *rabstol* -- scalar absolute residual tolerance.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_NO_MALLOC*  if the ARKStep memory was not allocated by the time-stepping module
      * *ARK_ILL_INPUT* if an argument had an illegal value (e.g. a negative tolerance).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeResStolerance` instead.


.. c:function:: int ARKStepResVtolerance(void* arkode_mem, N_Vector rabstol)

   This function specifies a vector of absolute residual tolerances.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *rabstol* -- vector containing the absolute residual
        tolerances for each solution component.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_NO_MALLOC*  if the ARKStep memory was not allocated by the time-stepping module
      * *ARK_ILL_INPUT* if an argument had an illegal value (e.g. a negative tolerance).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeResVtolerance` instead.


.. c:function:: int ARKStepResFtolerance(void* arkode_mem, ARKRwtFn rfun)

   This function specifies a user-supplied function *rfun* to compute
   the residual weight vector ``rwt``.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *rfun* -- the name of the function (of type :c:func:`ARKRwtFn()`)
        that implements the residual weight vector computation.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_NO_MALLOC*  if the ARKStep memory was not allocated by the time-stepping module

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeResFtolerance` instead.



.. _ARKODE.Usage.ARKStep.LinearSolvers:

Linear solver interface functions
-------------------------------------------

.. c:function:: int ARKStepSetLinearSolver(void* arkode_mem, SUNLinearSolver LS, SUNMatrix J)

   This function specifies the ``SUNLinearSolver`` object that ARKStep
   should use, as well as a template Jacobian ``SUNMatrix`` object (if
   applicable).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *LS* -- the ``SUNLinearSolver`` object to use.
      * *J* -- the template Jacobian ``SUNMatrix`` object to use (or
        ``NULL`` if not applicable).

   **Return value:**
      * *ARKLS_SUCCESS*   if successful
      * *ARKLS_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARKLS_MEM_FAIL*  if there was a memory allocation failure
      * *ARKLS_ILL_INPUT* if ARKLS is incompatible with the
        provided *LS* or *J* input objects, or the current
        ``N_Vector`` module.

   **Notes:**
      If *LS* is a matrix-free linear solver, then the *J*
      argument should be ``NULL``.

      If *LS* is a matrix-based linear solver, then the template Jacobian
      matrix *J* will be used in the solve process, so if additional
      storage is required within the ``SUNMatrix`` object (e.g. for
      factorization of a banded matrix), ensure that the input object is
      allocated with sufficient size (see the documentation of
      the particular SUNMATRIX type in the :numref:`SUNMatrix` for
      further information).

      When using sparse linear solvers, it is typically much more
      efficient to supply *J* so that it includes the full sparsity
      pattern of the Newton system matrices :math:`\mathcal{A} =
      M-\gamma J`, even if *J* itself has zeros in nonzero
      locations of :math:`M`.  The reasoning for this is
      that :math:`\mathcal{A}` is constructed in-place, on top of the
      user-specified values of *J*, so if the sparsity pattern in *J* is
      insufficient to store :math:`\mathcal{A}` then it will need to be
      resized internally by ARKStep.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetLinearSolver` instead.



.. _ARKODE.Usage.ARKStep.MassMatrixSolvers:

Mass matrix solver specification functions
-------------------------------------------


.. c:function:: int ARKStepSetMassLinearSolver(void* arkode_mem, SUNLinearSolver LS, SUNMatrix M, sunbooleantype time_dep)

   This function specifies the ``SUNLinearSolver`` object
   that ARKStep should use for mass matrix systems, as well as a
   template ``SUNMatrix`` object.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *LS* -- the ``SUNLinearSolver`` object to use.
      * *M* -- the template mass ``SUNMatrix`` object to use.
      * *time_dep* -- flag denoting whether the mass matrix depends on
        the independent variable (:math:`M = M(t)`) or not (:math:`M
        \ne M(t)`).  ``SUNTRUE`` indicates time-dependence of the
        mass matrix.

   **Return value:**
      * *ARKLS_SUCCESS*   if successful
      * *ARKLS_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARKLS_MEM_FAIL*  if there was a memory allocation failure
      * *ARKLS_ILL_INPUT* if ARKLS is incompatible with the
        provided *LS* or *M* input objects, or the current
        ``N_Vector`` module.

   **Notes:**
      If *LS* is a matrix-free linear solver, then the *M*
      argument should be ``NULL``.

      If *LS* is a matrix-based linear solver, then the template mass
      matrix *M* will be used in the solve process, so if additional
      storage is required within the ``SUNMatrix`` object (e.g. for
      factorization of a banded matrix), ensure that the input object is
      allocated with sufficient size.

      If called with *time_dep* set to ``SUNFALSE``, then the mass matrix is
      only computed and factored once (or when either :c:func:`ARKStepReInit()`
      or :c:func:`ARKStepResize()` are called), with the results reused
      throughout the entire ARKStep simulation.

      Unlike the system Jacobian, the system mass matrix is not approximated
      using finite-differences of any functions provided to ARKStep.  Hence,
      use of the a matrix-based *LS* requires the user to provide a
      mass-matrix constructor routine (see :c:type:`ARKLsMassFn` and
      :c:func:`ARKStepSetMassFn()`).

      Similarly, the system mass matrix-vector-product is not approximated
      using finite-differences of any functions provided to ARKStep.  Hence,
      use of a matrix-free *LS* requires the user to provide a
      mass-matrix-times-vector product routine (see
      :c:type:`ARKLsMassTimesVecFn` and :c:func:`ARKStepSetMassTimes()`).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMassLinearSolver` instead.


.. _ARKODE.Usage.ARKStep.NonlinearSolvers:

Nonlinear solver interface functions
-------------------------------------------


.. c:function:: int ARKStepSetNonlinearSolver(void* arkode_mem, SUNNonlinearSolver NLS)

   This function specifies the ``SUNNonlinearSolver`` object
   that ARKStep should use for implicit stage solves.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *NLS* -- the ``SUNNonlinearSolver`` object to use.

   **Return value:**
      * *ARK_SUCCESS*   if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_MEM_FAIL*  if there was a memory allocation failure
      * *ARK_ILL_INPUT* if ARKStep is incompatible with the
        provided *NLS* input object.

   **Notes:**
      ARKStep will use the Newton ``SUNNonlinearSolver`` module by
      default; a call to this routine replaces that module with the
      supplied *NLS* object.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetNonlinearSolver` instead.


.. _ARKODE.Usage.ARKStep.RootFinding:

Rootfinding initialization function
--------------------------------------


.. c:function:: int ARKStepRootInit(void* arkode_mem, int nrtfn, ARKRootFn g)

   Initializes a rootfinding problem to be solved during the
   integration of the ODE system.  It must be called after
   :c:func:`ARKStepCreate`, and before :c:func:`ARKStepEvolve()`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nrtfn* -- number of functions :math:`g_i`, an integer :math:`\ge` 0.
      * *g* -- name of user-supplied function, of type :c:func:`ARKRootFn()`,
        defining the functions :math:`g_i` whose roots are sought.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_MEM_FAIL*  if there was a memory allocation failure
      * *ARK_ILL_INPUT* if *nrtfn* is greater than zero but *g* = ``NULL``.

   **Notes:**
      To disable the rootfinding feature after it has already
      been initialized, or to free memory associated with ARKStep's
      rootfinding module, call *ARKStepRootInit* with *nrtfn = 0*.

      Similarly, if a new IVP is to be solved with a call to
      :c:func:`ARKStepReInit()`, where the new IVP has no rootfinding
      problem but the prior one did, then call *ARKStepRootInit* with
      *nrtfn = 0*.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeRootInit` instead.


.. _ARKODE.Usage.ARKStep.Integration:

ARKStep solver function
-------------------------


.. c:function:: int ARKStepEvolve(void* arkode_mem, sunrealtype tout, N_Vector yout, sunrealtype *tret, int itask)

   Integrates the ODE over an interval in :math:`t`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *tout* -- the next time at which a computed solution is desired.
      * *yout* -- the computed solution vector.
      * *tret* -- the time corresponding to *yout* (output).
      * *itask* -- a flag indicating the job of the solver for the next
        user step.

        The *ARK_NORMAL* option causes the solver to take internal
        steps until it has just overtaken a user-specified output
        time, *tout*, in the direction of integration,
        i.e. :math:`t_{n-1} <` *tout* :math:`\le t_{n}` for forward
        integration, or :math:`t_{n} \le` *tout* :math:`< t_{n-1}` for
        backward integration.  It will then compute an approximation
        to the solution :math:`y(tout)` by interpolation (as described
        in :numref:`ARKODE.Mathematics.Interpolation`).

        The *ARK_ONE_STEP* option tells the solver to only take a
        single internal step, :math:`y_{n-1} \to y_{n}`, and return the solution
        at that point, :math:`y_{n}`, in the vector *yout*.

   **Return value:**
      * *ARK_SUCCESS* if successful.
      * *ARK_ROOT_RETURN* if :c:func:`ARKStepEvolve()` succeeded, and
        found one or more roots.  If the number of root functions,
        *nrtfn*, is greater than 1, call
        :c:func:`ARKStepGetRootInfo()` to see which :math:`g_i` were
        found to have a root at (*\*tret*).
      * *ARK_TSTOP_RETURN* if :c:func:`ARKStepEvolve()` succeeded and
        returned at *tstop*.
      * *ARK_MEM_NULL* if the *arkode_mem* argument was ``NULL``.
      * *ARK_NO_MALLOC* if *arkode_mem* was not allocated.
      * *ARK_ILL_INPUT* if one of the inputs to
        :c:func:`ARKStepEvolve()` is illegal, or some other input to
        the solver was either illegal or missing.  Details will be
        provided in the error message.  Typical causes of this failure:

        (a) A component of the error weight vector became zero during
            internal time-stepping.

        (b) The linear solver initialization function (called by the
            user after calling :c:func:`ARKStepCreate`) failed to set
            the linear solver-specific *lsolve* field in
            *arkode_mem*.

        (c) A root of one of the root functions was found both at a
            point :math:`t` and also very near :math:`t`.

        (d) The initial condition violates the inequality constraints.

      * *ARK_TOO_MUCH_WORK* if the solver took *mxstep* internal steps
        but could not reach *tout*.  The default value for *mxstep* is
        *MXSTEP_DEFAULT = 500*.
      * *ARK_TOO_MUCH_ACC* if the solver could not satisfy the accuracy
        demanded by the user for some internal step.
      * *ARK_ERR_FAILURE* if error test failures occurred either too many
        times (*ark_maxnef*) during one internal time step or occurred
        with :math:`|h| = h_{min}`.
      * *ARK_CONV_FAILURE* if either convergence test failures occurred
        too many times (*ark_maxncf*) during one internal time step or
        occurred with :math:`|h| = h_{min}`.
      * *ARK_LINIT_FAIL* if the linear solver's initialization
        function failed.
      * *ARK_LSETUP_FAIL* if the linear solver's setup routine failed in
        an unrecoverable manner.
      * *ARK_LSOLVE_FAIL* if the linear solver's solve routine failed in
        an unrecoverable manner.
      * *ARK_MASSINIT_FAIL* if the mass matrix solver's
        initialization function failed.
      * *ARK_MASSSETUP_FAIL* if the mass matrix solver's setup routine
        failed.
      * *ARK_MASSSOLVE_FAIL* if the mass matrix solver's solve routine
        failed.
      * *ARK_VECTOROP_ERR* a vector operation error occurred.

   **Notes:**
      The input vector *yout* can use the same memory as the
      vector *y0* of initial conditions that was passed to
      :c:func:`ARKStepCreate`.

      In *ARK_ONE_STEP* mode, *tout* is used only on the first call, and
      only to get the direction and a rough scale of the independent
      variable.

      All failure return values are negative and so testing the return argument
      for negative values will trap all :c:func:`ARKStepEvolve()` failures.

      Since interpolation may reduce the accuracy in the reported
      solution, if full method accuracy is desired the user should issue
      a call to :c:func:`ARKStepSetStopTime()` before the call to
      :c:func:`ARKStepEvolve()` to specify a fixed stop time to
      end the time step and return to the user.  Upon return from
      :c:func:`ARKStepEvolve()`, a copy of the internal solution
      :math:`y_{n}` will be returned in the vector *yout*.  Once the
      integrator returns at a *tstop* time, any future testing for
      *tstop* is disabled (and can be re-enabled only though a new call
      to :c:func:`ARKStepSetStopTime()`).

      On any error return in which one or more internal steps were taken
      by :c:func:`ARKStepEvolve()`, the returned values of *tret* and
      *yout* correspond to the farthest point reached in the integration.
      On all other error returns, *tret* and *yout* are left unchanged
      from those provided to the routine.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeEvolve` instead.


.. _ARKODE.Usage.ARKStep.OptionalInputs:

Optional input functions
-------------------------


.. _ARKODE.Usage.ARKStep.ARKStepInputTable:

Optional inputs for ARKStep
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. c:function:: int ARKStepSetDefaults(void* arkode_mem)

   Resets all optional input parameters to ARKStep's original
   default values.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Does not change the *user_data* pointer or any
      parameters within the specified time-stepping module.

      Also leaves alone any data structures or options related to
      root-finding (those can be reset using :c:func:`ARKStepRootInit()`).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetDefaults` instead.


.. c:function:: int ARKStepSetInterpolantType(void* arkode_mem, int itype)

   .. deprecated:: 6.1.0

      This function is now a wrapper to :c:func:`ARKodeSetInterpolantType`, see
      the documentation for that function instead.


.. c:function:: int ARKStepSetInterpolantDegree(void* arkode_mem, int degree)

   Specifies the degree of the polynomial interpolant
   used for dense output (i.e. interpolation of solution output values
   and implicit method predictors).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *degree* -- requested polynomial degree.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory or interpolation module are ``NULL``
      * *ARK_INTERP_FAIL* if this is called after :c:func:`ARKStepEvolve()`
      * *ARK_ILL_INPUT* if an argument had an illegal value or the
        interpolation module has already been initialized

   **Notes:**
      Allowed values are between 0 and 5.

      This routine should be called *after* :c:func:`ARKStepCreate` and *before*
      :c:func:`ARKStepEvolve()`. After the first call to :c:func:`ARKStepEvolve()`
      the interpolation degree may not be changed without first calling
      :c:func:`ARKStepReInit()`.

      If a user calls both this routine and :c:func:`ARKStepSetInterpolantType()`, then
      :c:func:`ARKStepSetInterpolantType()` must be called first.

      Since the accuracy of any polynomial interpolant is limited by the
      accuracy of the time-step solutions on which it is based, the *actual*
      polynomial degree that is used by ARKStep will be the minimum of
      :math:`q-1` and the input *degree*, for :math:`q > 1` where :math:`q` is
      the order of accuracy for the time integration method.

      .. versionchanged:: 5.5.1

         When :math:`q=1`, a linear interpolant is the default to ensure values
         obtained by the integrator are returned at the ends of the time
         interval.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetInterpolantDegree` instead.


.. c:function:: int ARKStepSetDenseOrder(void* arkode_mem, int dord)

   .. deprecated:: 5.2.0

      Use :c:func:`ARKodeSetInterpolantDegree` instead.


.. c:function:: int ARKStepSetDiagnostics(void* arkode_mem, FILE* diagfp)

   Specifies the file pointer for a diagnostics file where
   all ARKStep step adaptivity and solver information is written.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *diagfp* -- pointer to the diagnostics output file.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This parameter can be ``stdout`` or ``stderr``, although the
      suggested approach is to specify a pointer to a unique file opened
      by the user and returned by ``fopen``.  If not called, or if called
      with a ``NULL`` file pointer, all diagnostics output is disabled.

      When run in parallel, only one process should set a non-NULL value
      for this pointer, since statistics from all processes would be
      identical.

   .. deprecated:: 5.2.0

      Use :c:func:`SUNLogger_SetInfoFilename` instead.


.. c:function:: int ARKStepSetFixedStep(void* arkode_mem, sunrealtype hfixed)

   Disables time step adaptivity within ARKStep, and specifies the
   fixed time step size to use for the following internal step(s).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *hfixed* -- value of the fixed step size to use.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Pass 0.0 to return ARKStep to the default (adaptive-step) mode.

      Use of this function is not generally recommended, since it gives no
      assurance of the validity of the computed solutions.  It is
      primarily provided for code-to-code verification testing purposes.

      When using :c:func:`ARKStepSetFixedStep()`, any values provided to
      the functions
      :c:func:`ARKStepSetInitStep()`,
      :c:func:`ARKStepSetAdaptivityFn()`,
      :c:func:`ARKStepSetMaxErrTestFails()`,
      :c:func:`ARKStepSetAdaptivityMethod()`,
      :c:func:`ARKStepSetCFLFraction()`,
      :c:func:`ARKStepSetErrorBias()`,
      :c:func:`ARKStepSetFixedStepBounds()`,
      :c:func:`ARKStepSetMaxCFailGrowth()`,
      :c:func:`ARKStepSetMaxEFailGrowth()`,
      :c:func:`ARKStepSetMaxFirstGrowth()`,
      :c:func:`ARKStepSetMaxGrowth()`,
      :c:func:`ARKStepSetMinReduction()`,
      :c:func:`ARKStepSetSafetyFactor()`,
      :c:func:`ARKStepSetSmallNumEFails()`,
      :c:func:`ARKStepSetStabilityFn()`, and
      :c:func:`ARKStepSetAdaptController()`
      will be ignored, since temporal adaptivity is disabled.

      If both :c:func:`ARKStepSetFixedStep()` and
      :c:func:`ARKStepSetStopTime()` are used, then the fixed step size
      will be used for all steps until the final step preceding the
      provided stop time (which may be shorter).  To resume use of the
      previous fixed step size, another call to
      :c:func:`ARKStepSetFixedStep()` must be made prior to calling
      :c:func:`ARKStepEvolve()` to resume integration.

      It is *not* recommended that :c:func:`ARKStepSetFixedStep()` be used
      in concert with :c:func:`ARKStepSetMaxStep()` or
      :c:func:`ARKStepSetMinStep()`, since at best those latter two
      routines will provide no useful information to the solver, and at
      worst they may interfere with the desired fixed step size.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetFixedStep` instead.



.. c:function:: int ARKStepSetInitStep(void* arkode_mem, sunrealtype hin)

   Specifies the initial time step size ARKStep should use after
   initialization, re-initialization, or resetting.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *hin* -- value of the initial step to be attempted :math:`(\ne 0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Pass 0.0 to use the default value.

      By default, ARKStep estimates the initial step size to be
      :math:`h = \sqrt{\dfrac{2}{\left\| \ddot{y}\right\|}}`, where
      :math:`\ddot{y}` is estimate of the second derivative of the solution
      at :math:`t_0`.

      This routine will also reset the step size and error history.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetInitStep` instead.


.. c:function:: int ARKStepSetMaxHnilWarns(void* arkode_mem, int mxhnil)

   Specifies the maximum number of messages issued by the
   solver to warn that :math:`t+h=t` on the next internal step, before
   ARKStep will instead return with an error.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *mxhnil* -- maximum allowed number of warning messages :math:`(>0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The default value is 10; set *mxhnil* to zero to specify
      this default.

      A negative value indicates that no warning messages should be issued.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxHnilWarns` instead.


.. c:function:: int ARKStepSetMaxNumSteps(void* arkode_mem, long int mxsteps)

   Specifies the maximum number of steps to be taken by the
   solver in its attempt to reach the next output time, before ARKStep
   will return with an error.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *mxsteps* -- maximum allowed number of internal steps.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Passing *mxsteps* = 0 results in ARKStep using the
      default value (500).

      Passing *mxsteps* < 0 disables the test (not recommended).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxNumSteps` instead.


.. c:function:: int ARKStepSetMaxStep(void* arkode_mem, sunrealtype hmax)

   Specifies the upper bound on the magnitude of the time step size.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *hmax* -- maximum absolute value of the time step size :math:`(\ge 0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Pass *hmax* :math:`\le 0.0` to set the default value of :math:`\infty`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxStep` instead.


.. c:function:: int ARKStepSetMinStep(void* arkode_mem, sunrealtype hmin)

   Specifies the lower bound on the magnitude of the time step size.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *hmin* -- minimum absolute value of the time step size :math:`(\ge 0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Pass *hmin* :math:`\le 0.0` to set the default value of 0.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMinStep` instead.


.. c:function:: int ARKStepSetStopTime(void* arkode_mem, sunrealtype tstop)

   Specifies the value of the independent variable
   :math:`t` past which the solution is not to proceed.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *tstop* -- stopping time for the integrator.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The default is that no stop time is imposed.

      Once the integrator returns at a stop time, any future testing for
      ``tstop`` is disabled (and can be re-enabled only though a new call to
      :c:func:`ARKStepSetStopTime`).

      A stop time not reached before a call to :c:func:`ARKStepReInit` or
      :c:func:`ARKStepReset` will remain active but can be disabled by calling
      :c:func:`ARKStepClearStopTime`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetStopTime` instead.


.. c:function:: int ARKStepSetInterpolateStopTime(void* arkode_mem, sunbooleantype interp)

   Specifies that the output solution should be interpolated when the current
   :math:`t` equals the specified ``tstop`` (instead of merely copying the
   internal solution :math:`y_n`).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *interp* -- flag indicating to use interpolation (1) or copy (0).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``

   .. versionadded:: 5.6.0

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetInterpolateStopTime` instead.


.. c:function:: int ARKStepClearStopTime(void* arkode_mem)

   Disables the stop time set with :c:func:`ARKStepSetStopTime`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``

   **Notes:**
      The stop time can be re-enabled though a new call to
      :c:func:`ARKStepSetStopTime`.

   .. versionadded:: 5.5.1

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeClearStopTime` instead.


.. c:function:: int ARKStepSetUserData(void* arkode_mem, void* user_data)

   Specifies the user data block *user_data* and
   attaches it to the main ARKStep memory block.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *user_data* -- pointer to the user data.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      If specified, the pointer to *user_data* is passed to all
      user-supplied functions for which it is an argument; otherwise
      ``NULL`` is passed.

      If *user_data* is needed in user preconditioner functions, the call to
      this function must be made *before* any calls to
      :c:func:`ARKStepSetLinearSolver()` and/or :c:func:`ARKStepSetMassLinearSolver()`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetUserData` instead.


.. c:function:: int ARKStepSetMaxErrTestFails(void* arkode_mem, int maxnef)

   Specifies the maximum number of error test failures
   permitted in attempting one step, before returning with an error.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *maxnef* -- maximum allowed number of error test failures :math:`(>0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The default value is 7; set *maxnef* :math:`\le 0`
      to specify this default.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxErrTestFails` instead.


.. c:function:: int ARKStepSetOptimalParams(void* arkode_mem)

   Sets all adaptivity and solver parameters to our "best
   guess" values for a given integration method type (ERK, DIRK, ARK) and
   a given method order.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Should only be called after the method order and integration
      method have been set.  The "optimal" values resulted from repeated testing
      of ARKStep's solvers on a variety of training problems.  However,
      all problems are different, so these values may not be optimal for
      all users.

   .. deprecated:: 6.1.0

      Adjust solver parameters individually instead.  For reference, this routine
      sets the following non-default parameters:

      * Explicit methods:

        * :c:func:`SUNAdaptController_PI` with :c:func:`SUNAdaptController_SetErrorBias` of 1.2 and :c:func:`SUNAdaptController_SetParams_PI` of :math:`k_1=0.8` and :math:`k_2=-0.31`

        * :c:func:`ARKodeSetSafetyFactor` of 0.99

        * :c:func:`ARKodeSetMaxGrowth` of 25.0

        * :c:func:`ARKodeSetMaxEFailGrowth` of 0.3

      * Implicit methods:

        * Order 3:

          * :c:func:`SUNAdaptController_I` with :c:func:`SUNAdaptController_SetErrorBias` of 1.9

          * :c:func:`ARKodeSetSafetyFactor` of 0.957

          * :c:func:`ARKodeSetMaxGrowth` of 17.6

          * :c:func:`ARKodeSetMaxEFailGrowth` of 0.45

          * :c:func:`ARKodeSetNonlinConvCoef` of 0.22

          * :c:func:`ARKodeSetNonlinCRDown` of 0.17

          * :c:func:`ARKodeSetNonlinRDiv` of 2.3

          * :c:func:`ARKodeSetDeltaGammaMax` of 0.19

        * Order 4:

          * :c:func:`SUNAdaptController_PID` with :c:func:`SUNAdaptController_SetErrorBias` of 1.2 and :c:func:`SUNAdaptController_SetParams_PID` of :math:`k_1=0.535`, :math:`k_2=-0.209`, and :math:`k_3=0.148`

          * :c:func:`ARKodeSetSafetyFactor` of 0.988

          * :c:func:`ARKodeSetMaxGrowth` of 31.5

          * :c:func:`ARKodeSetMaxEFailGrowth` of 0.33

          * :c:func:`ARKodeSetNonlinConvCoef` of 0.24

          * :c:func:`ARKodeSetNonlinCRDown` of 0.26

          * :c:func:`ARKodeSetNonlinRDiv` of 2.3

          * :c:func:`ARKodeSetDeltaGammaMax` of 0.16

          * :c:func:`ARKodeSetLSetupFrequency` of 31

        * Order 5:

          * :c:func:`SUNAdaptController_PID` with :c:func:`SUNAdaptController_SetErrorBias` of 3.3 and :c:func:`SUNAdaptController_SetParams_PID` of :math:`k_1=0.56`, :math:`k_2=-0.338`, and :math:`k_3=0.14`

          * :c:func:`ARKodeSetSafetyFactor` of 0.937

          * :c:func:`ARKodeSetMaxGrowth` of 22.0

          * :c:func:`ARKodeSetMaxEFailGrowth` of 0.44

          * :c:func:`ARKodeSetNonlinConvCoef` of 0.25

          * :c:func:`ARKodeSetNonlinCRDown` of 0.4

          * :c:func:`ARKodeSetNonlinRDiv` of 2.3

          * :c:func:`ARKodeSetDeltaGammaMax` of 0.32

          * :c:func:`ARKodeSetLSetupFrequency` of 31

      * ImEx methods:

        * Order 2:

          * :c:func:`ARKodeSetNonlinConvCoef` of 0.001

          * :c:func:`ARKodeSetMaxNonlinIters` of 5

        * Order 3:

          * :c:func:`SUNAdaptController_PID` with :c:func:`SUNAdaptController_SetErrorBias` of 1.42 and :c:func:`SUNAdaptController_SetParams_PID` of :math:`k_1=0.54`, :math:`k_2=-0.36`, and :math:`k_3=0.14`

          * :c:func:`ARKodeSetSafetyFactor` of 0.965

          * :c:func:`ARKodeSetMaxGrowth` of 28.7

          * :c:func:`ARKodeSetMaxEFailGrowth` of 0.46

          * :c:func:`ARKodeSetNonlinConvCoef` of 0.22

          * :c:func:`ARKodeSetNonlinCRDown` of 0.17

          * :c:func:`ARKodeSetNonlinRDiv` of 2.3

          * :c:func:`ARKodeSetDeltaGammaMax` of 0.19

          * :c:func:`ARKodeSetLSetupFrequency` of 60

        * Order 4:

          * :c:func:`SUNAdaptController_PID` with :c:func:`SUNAdaptController_SetErrorBias` of 1.35 and :c:func:`SUNAdaptController_SetParams_PID` of :math:`k_1=0.543`, :math:`k_2=-0.297`, and :math:`k_3=0.14`

          * :c:func:`ARKodeSetSafetyFactor` of 0.97

          * :c:func:`ARKodeSetMaxGrowth` of 25.0

          * :c:func:`ARKodeSetMaxEFailGrowth` of 0.47

          * :c:func:`ARKodeSetNonlinConvCoef` of 0.24

          * :c:func:`ARKodeSetNonlinCRDown` of 0.26

          * :c:func:`ARKodeSetNonlinRDiv` of 2.3

          * :c:func:`ARKodeSetDeltaGammaMax` of 0.16

          * :c:func:`ARKodeSetLSetupFrequency` of 31

        * Order 5:

          * :c:func:`SUNAdaptController_PI` with :c:func:`SUNAdaptController_SetErrorBias` of 1.15 and :c:func:`SUNAdaptController_SetParams_PI` of :math:`k_1=0.8` and :math:`k_2=-0.35`

          * :c:func:`ARKodeSetSafetyFactor` of 0.993

          * :c:func:`ARKodeSetMaxGrowth` of 28.5

          * :c:func:`ARKodeSetMaxEFailGrowth` of 0.3

          * :c:func:`ARKodeSetNonlinConvCoef` of 0.25

          * :c:func:`ARKodeSetNonlinCRDown` of 0.4

          * :c:func:`ARKodeSetNonlinRDiv` of 2.3

          * :c:func:`ARKodeSetDeltaGammaMax` of 0.32

          * :c:func:`ARKodeSetLSetupFrequency` of 31


.. c:function:: int ARKStepSetConstraints(void* arkode_mem, N_Vector constraints)

   Specifies a vector defining inequality constraints for each component of the
   solution vector :math:`y`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *constraints* -- vector of constraint flags. Each component specifies
        the type of solution constraint:

        .. math::

           \texttt{constraints[i]} = \left\{ \begin{array}{rcl}
               0.0  &\Rightarrow\;& \text{no constraint is imposed on}\; y_i,\\
               1.0  &\Rightarrow\;& y_i \geq 0,\\
              -1.0  &\Rightarrow\;& y_i \leq 0,\\
               2.0  &\Rightarrow\;& y_i > 0,\\
              -2.0  &\Rightarrow\;& y_i < 0.\\
              \end{array}\right.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if the constraints vector contains illegal values

   **Notes:**
      The presence of a non-``NULL`` constraints vector that is not 0.0
      in all components will cause constraint checking to be performed. However, a
      call with 0.0 in all components of ``constraints`` will result in an illegal
      input return. A ``NULL`` constraints vector will disable constraint checking.

      After a call to :c:func:`ARKStepResize()` inequality constraint checking
      will be disabled and a call to :c:func:`ARKStepSetConstraints()` is
      required to re-enable constraint checking.

      Since constraint-handling is performed through cutting time steps that would
      violate the constraints, it is possible that this feature will cause some
      problems to fail due to an inability to enforce constraints even at the
      minimum time step size.  Additionally, the features :c:func:`ARKStepSetConstraints()`
      and :c:func:`ARKStepSetFixedStep()` are incompatible, and should not be used
      simultaneously.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetConstraints` instead.


.. c:function:: int ARKStepSetMaxNumConstrFails(void* arkode_mem, int maxfails)

   Specifies the maximum number of constraint failures in a step before ARKStep
   will return with an error.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *maxfails* -- maximum allowed number of constrain failures.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``

   **Notes:**
      Passing *maxfails* <= 0 results in ARKStep using the
      default value (10).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxNumConstrFails` instead.


.. _ARKODE.Usage.ARKStep.ARKStepMethodInputTable:

Optional inputs for IVP method selection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. cssclass:: table-bordered

========================================  =================================  ==============
Optional input                            Function name                      Default
========================================  =================================  ==============
Set integrator method order               :c:func:`ARKStepSetOrder()`        4
Specify implicit/explicit problem         :c:func:`ARKStepSetImEx()`         ``SUNTRUE``
Specify explicit problem                  :c:func:`ARKStepSetExplicit()`     ``SUNFALSE``
Specify implicit problem                  :c:func:`ARKStepSetImplicit()`     ``SUNFALSE``
Set additive RK tables                    :c:func:`ARKStepSetTables()`       internal
Set additive RK tables via their numbers  :c:func:`ARKStepSetTableNum()`     internal
Set additive RK tables via their names    :c:func:`ARKStepSetTableName()`    internal
========================================  =================================  ==============



.. c:function:: int ARKStepSetOrder(void* arkode_mem, int ord)

   Specifies the order of accuracy for the ARK/DIRK/ERK integration
   method.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *ord* -- requested order of accuracy.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      For explicit methods, the allowed values are :math:`2 \le`
      *ord* :math:`\le 8`.  For implicit methods, the allowed values are
      :math:`2\le` *ord* :math:`\le 5`, and for ImEx methods the allowed
      values are :math:`2 \le` *ord* :math:`\le 5`.  Any illegal input
      will result in the default value of 4.

      Since *ord* affects the memory requirements for the internal
      ARKStep memory block, it cannot be changed after the first call to
      :c:func:`ARKStepEvolve()`, unless :c:func:`ARKStepReInit()` is called.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetOrder` instead.


.. c:function:: int ARKStepSetImEx(void* arkode_mem)

   Specifies that both the implicit and explicit portions
   of problem are enabled, and to use an additive Runge--Kutta method.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This is automatically deduced when neither of the function
      pointers *fe* or *fi* passed to :c:func:`ARKStepCreate` are
      ``NULL``, but may be set directly by the user if desired.



.. c:function:: int ARKStepSetExplicit(void* arkode_mem)

   Specifies that the implicit portion of problem is disabled,
   and to use an explicit RK method.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This is automatically deduced when the function pointer `fi`
      passed to :c:func:`ARKStepCreate` is ``NULL``, but may be set
      directly by the user if desired.

      If the problem is posed in explicit form, i.e. :math:`\dot{y} =
      f(t,y)`, then we recommend that the ERKStep time-stepper module be
      used instead.


.. c:function:: int ARKStepSetImplicit(void* arkode_mem)

   Specifies that the explicit portion of problem is disabled,
   and to use a diagonally implicit RK method.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This is automatically deduced when the function pointer `fe`
      passed to :c:func:`ARKStepCreate` is ``NULL``, but may be set
      directly by the user if desired.



.. c:function:: int ARKStepSetTables(void* arkode_mem, int q, int p, ARKodeButcherTable Bi, ARKodeButcherTable Be)

   Specifies a customized Butcher table (or pair) for the ERK, DIRK, or ARK method.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *q* -- global order of accuracy for the ARK method.
      * *p* -- global order of accuracy for the embedded ARK method.
      * *Bi* -- the Butcher table for the implicit RK method.
      * *Be* -- the Butcher table for the explicit RK method.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      For a description of the :c:type:`ARKodeButcherTable` type and related
      functions for creating Butcher tables, see :numref:`ARKodeButcherTable`.

      To set an explicit table, *Bi* must be ``NULL``.  This automatically calls
      :c:func:`ARKStepSetExplicit()`.  However, if the problem is posed
      in explicit form, i.e. :math:`\dot{y} = f(t,y)`, then we recommend
      that the ERKStep time-stepper module be used instead of ARKStep.

      To set an implicit table, *Be* must be ``NULL``.  This automatically calls
      :c:func:`ARKStepSetImplicit()`.

      If both *Bi* and *Be* are provided, this routine automatically calls
      :c:func:`ARKStepSetImEx()`.

      When only one table is provided (i.e., *Bi* or *Be* is ``NULL``) then the
      input values of *q* and *p* are ignored and the global order of the method
      and embedding (if applicable) are obtained from the Butcher table
      structures. If both *Bi* and *Be* are non-NULL (e.g, an ImEx method is
      provided) then the input values of *q* and *p* are used as the order of the
      ARK method may be less than the orders of the individual tables. No error
      checking is performed to ensure that either *p* or *q* correctly describe the
      coefficients that were input.

      Error checking is subsequently performed at ARKStep initialization to ensure
      that *Bi* and *Be* (if non-NULL) specify DIRK and ERK methods, respectively.
      Specifically, the *A* member of *Bi* must be lower triangular with at least
      one nonzero value on the diagonal, and the *A* member of *Be* must be strictly
      lower triangular.  When both *Bi* and *Be* are non-NULL, they must agree on
      the number of internal stages, i.e., the *stages* members of both structures
      must match.

      If the inputs *Bi* or *Be* do not contain an embedding (when the
      corresponding explicit or implicit table is non-NULL), the user *must* call
      :c:func:`ARKStepSetFixedStep()` to enable fixed-step mode and set the
      desired time step size.

   **Warning:**
      This should not be used with :c:func:`ARKodeSetOrder`.



.. c:function:: int ARKStepSetTableNum(void* arkode_mem, ARKODE_DIRKTableID itable, ARKODE_ERKTableID etable)

   Indicates to use specific built-in Butcher tables for the ERK, DIRK
   or ARK method.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *itable* -- index of the DIRK Butcher table.
      * *etable* -- index of the ERK Butcher table.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The allowable values for both the *itable* and *etable* arguments
      corresponding to built-in tables may be found in :numref:`Butcher`.

      To choose an explicit table, set *itable* to a negative value.  This
      automatically calls :c:func:`ARKStepSetExplicit()`.  However, if
      the problem is posed in explicit form, i.e. :math:`\dot{y} =
      f(t,y)`, then we recommend that the ERKStep time-stepper module be
      used instead of ARKStep.

      To select an implicit table, set *etable* to a negative value.
      This automatically calls :c:func:`ARKStepSetImplicit()`.

      If both *itable* and *etable* are non-negative, then these should
      match an existing implicit/explicit pair, listed in
      :numref:`Butcher.additive`.  This automatically calls
      :c:func:`ARKStepSetImEx()`.

      In all cases, error-checking is performed to ensure that the tables
      exist.

   **Warning:**
      This should not be used with :c:func:`ARKodeSetOrder`.




.. c:function:: int ARKStepSetTableName(void* arkode_mem, const char *itable, const char *etable)

   Indicates to use specific built-in Butcher tables for the ERK, DIRK
   or ARK method.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *itable* -- name of the DIRK Butcher table.
      * *etable* -- name of the ERK Butcher table.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The allowable values for both the *itable* and *etable* arguments
      corresponding to built-in tables may be found in :numref:`Butcher`.
      This function is case sensitive.

      To choose an explicit table, set *itable* to ``"ARKODE_DIRK_NONE"``.
      This automatically calls :c:func:`ARKStepSetExplicit()`.  However,
      if the problem is posed in explicit form, i.e. :math:`\dot{y} =
      f(t,y)`, then we recommend that the ERKStep time-stepper module be
      used instead of ARKStep.

      To select an implicit table, set *etable* to ``"ARKODE_ERK_NONE"``.
      This automatically calls :c:func:`ARKStepSetImplicit()`.

      If both *itable* and *etable* are not none, then these should match
      an existing implicit/explicit pair, listed in
      :numref:`Butcher.additive`.  This automatically calls
      :c:func:`ARKStepSetImEx()`.

      In all cases, error-checking is performed to ensure that the tables
      exist.

   **Warning:**
      This should not be used with :c:func:`ARKodeSetOrder`.




.. _ARKODE.Usage.ARKStep.ARKStepAdaptivityInputTable:

Optional inputs for time step adaptivity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. c:function:: int ARKStepSetAdaptController(void* arkode_mem, SUNAdaptController C)

   Sets a user-supplied time-step controller object.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *C* -- user-supplied time adaptivity controller.  If ``NULL`` then the I controller will be created (see :numref:`SUNAdaptController.Soderlind`).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_MEM_FAIL* if *C* was ``NULL`` and the I controller could not be allocated.

   .. versionadded:: 5.7.0

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetAdaptController` instead.

   .. versionchanged:: 6.3.0

      The default controller was changed from PID to I.


.. c:function:: int ARKStepSetAdaptivityFn(void* arkode_mem, ARKAdaptFn hfun, void* h_data)

   Sets a user-supplied time-step adaptivity function.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *hfun* -- name of user-supplied adaptivity function.
      * *h_data* -- pointer to user data passed to *hfun* every time
        it is called.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This function should focus on accuracy-based time step
      estimation; for stability based time steps the function
      :c:func:`ARKStepSetStabilityFn()` should be used instead.


   .. deprecated:: 5.7.0

      Use the SUNAdaptController infrastructure instead (see :numref:`SUNAdaptController.Description`).



.. c:function:: int ARKStepSetAdaptivityMethod(void* arkode_mem, int imethod, int idefault, int pq, sunrealtype* adapt_params)

   Specifies the method (and associated parameters) used for time step adaptivity.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *imethod* -- accuracy-based adaptivity method choice
        (0 :math:`\le` `imethod` :math:`\le` 5):
        0 is PID, 1 is PI, 2 is I, 3 is explicit Gustafsson, 4 is
        implicit Gustafsson, and 5 is the ImEx Gustafsson.
      * *idefault* -- flag denoting whether to use default adaptivity
        parameters (1), or that they will be supplied in the
        *adapt_params* argument (0).
      * *pq* -- flag denoting whether to use the embedding order of
        accuracy *p* (0), the method order of accuracy *q* (1), or the
        minimum of the two (any input not equal to 0 or 1)
        within the adaptivity algorithm.  *p* is the default.
      * *adapt_params[0]* -- :math:`k_1` parameter within accuracy-based adaptivity algorithms.
      * *adapt_params[1]* -- :math:`k_2` parameter within accuracy-based adaptivity algorithms.
      * *adapt_params[2]* -- :math:`k_3` parameter within accuracy-based adaptivity algorithms.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      If custom parameters are supplied, they will be checked
      for validity against published stability intervals.  If other
      parameter values are desired, it is recommended to instead provide
      a custom function through a call to :c:func:`ARKStepSetAdaptivityFn()`.

      .. versionchanged:: 5.7.0

         Prior to version 5.7.0, any nonzero value for *pq* would result in use of the
         embedding order of accuracy.


   .. deprecated:: 5.7.0

      Use the SUNAdaptController infrastructure instead (see :numref:`SUNAdaptController.Description`).



.. c:function:: int ARKStepSetAdaptivityAdjustment(void* arkode_mem, int adjust)

   Called by a user to adjust the method order supplied to the temporal adaptivity
   controller.  For example, if the user expects order reduction due to problem stiffness,
   they may request that the controller assume a reduced order of accuracy for the method
   by specifying a value :math:`adjust < 0`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *adjust* -- adjustment factor (default is 0).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This should be called prior to calling :c:func:`ARKStepEvolve()`, and can only be
      reset following a call to :c:func:`ARKStepReInit()`.

   .. versionadded:: 5.7.0

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetAdaptivityAdjustment` instead.

   .. versionchanged:: 6.3.0

      The default value was changed from -1 to 0



.. c:function:: int ARKStepSetCFLFraction(void* arkode_mem, sunrealtype cfl_frac)

   Specifies the fraction of the estimated explicitly stable step to use.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *cfl_frac* -- maximum allowed fraction of explicitly stable step (default is 0.5).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any non-positive parameter will imply a reset to the default
      value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetCFLFraction` instead.


.. c:function:: int ARKStepSetErrorBias(void* arkode_mem, sunrealtype bias)

   Specifies the bias to be applied to the error estimates within
   accuracy-based adaptivity strategies.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *bias* -- bias applied to error in accuracy-based time
        step estimation (default is 1.0).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any value below 1.0 will imply a reset to the default value.

      If both this and one of :c:func:`ARKStepSetAdaptivityMethod` or
      :c:func:`ARKStepSetAdaptController` will be called, then this routine must be called
      *second*.

   .. deprecated:: 5.7.0

      Use the SUNAdaptController infrastructure instead (see :numref:`SUNAdaptController.Description`).
      
   .. versionchanged:: 6.3.0

      The default value was changed from 1.5 to 1.0


.. c:function:: int ARKStepSetFixedStepBounds(void* arkode_mem, sunrealtype lb, sunrealtype ub)

   Specifies the step growth interval in which the step size will remain unchanged.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *lb* -- lower bound on window to leave step size fixed (default is 1.0).
      * *ub* -- upper bound on window to leave step size fixed (default is 1.0).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any interval *not* containing 1.0 will imply a reset to the default values.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetFixedStepBounds` instead.
      
   .. versionchanged:: 6.3.0

      The default upper bound was changed from 1.5 to 1.0


.. c:function:: int ARKStepSetMaxCFailGrowth(void* arkode_mem, sunrealtype etacf)

   Specifies the maximum step size growth factor upon an algebraic
   solver convergence failure on a stage solve within a step, :math:`\eta_{cf}` from
   :numref:`ARKODE.Mathematics.Error.Nonlinear`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *etacf* -- time step reduction factor on a nonlinear solver
        convergence failure (default is 0.25).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any value outside the interval :math:`(0,1]` will imply a reset to the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxCFailGrowth` instead.


.. c:function:: int ARKStepSetMaxEFailGrowth(void* arkode_mem, sunrealtype etamxf)

   Specifies the maximum step size growth factor upon multiple successive
   accuracy-based error failures in the solver.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *etamxf* -- time step reduction factor on multiple error fails (default is 0.3).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any value outside the interval :math:`(0,1]` will imply a reset to the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxEFailGrowth` instead.


.. c:function:: int ARKStepSetMaxFirstGrowth(void* arkode_mem, sunrealtype etamx1)

   Specifies the maximum allowed growth factor in step size following the very
   first integration step.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *etamx1* -- maximum allowed growth factor after the first time
        step (default is 10000.0).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any value :math:`\le 1.0` will imply a reset to the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxFirstGrowth` instead.


.. c:function:: int ARKStepSetMaxGrowth(void* arkode_mem, sunrealtype mx_growth)

   Specifies the maximum allowed growth factor in step size between
   consecutive steps in the integration process.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *mx_growth* -- maximum allowed growth factor between consecutive time steps (default is 20.0).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any value :math:`\le 1.0` will imply a reset to the default
      value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxGrowth` instead.


.. c:function:: int ARKStepSetMinReduction(void* arkode_mem, sunrealtype eta_min)

   Specifies the minimum allowed reduction factor in step size between
   step attempts, resulting from a temporal error failure in the integration
   process.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *eta_min* -- minimum allowed reduction factor in time step after an error
        test failure (default is 0.1).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any value outside the interval :math:`(0,1)` will imply a reset to
      the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMinReduction` instead.


.. c:function:: int ARKStepSetSafetyFactor(void* arkode_mem, sunrealtype safety)

   Specifies the safety factor to be applied to the accuracy-based
   estimated step.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *safety* -- safety factor applied to accuracy-based time step (default is 0.9).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any value :math:`\le 0` will imply a reset to the default
      value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetSafetyFactor` instead.
      
   .. versionchanged:: 6.3.0

      The default default was changed from 0.96 to 0.9. The maximum value is now
      exactly 1.0 rather than strictly less than 1.0.


.. c:function:: int ARKStepSetSmallNumEFails(void* arkode_mem, int small_nef)

   Specifies the threshold for "multiple" successive error failures
   before the *etamxf* parameter from
   :c:func:`ARKStepSetMaxEFailGrowth()` is applied.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *small_nef* -- bound to determine 'multiple' for *etamxf* (default is 2).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any value :math:`\le 0` will imply a reset to the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetSmallNumEFails` instead.


.. c:function:: int ARKStepSetStabilityFn(void* arkode_mem, ARKExpStabFn EStab, void* estab_data)

   Sets the problem-dependent function to estimate a stable
   time step size for the explicit portion of the ODE system.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *EStab* -- name of user-supplied stability function.
      * *estab_data* -- pointer to user data passed to *EStab* every time
        it is called.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This function should return an estimate of the absolute
      value of the maximum stable time step for the explicit portion of
      the ODE system.  It is not required, since accuracy-based
      adaptivity may be sufficient for retaining stability, but this can
      be quite useful for problems where the explicit right-hand side
      function :math:`f^E(t,y)` contains stiff terms.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetStabilityFn` instead.



.. _ARKODE.Usage.ARKStep.ARKStepSolverInputTable:

Optional inputs for implicit stage solves
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. c:function:: int ARKStepSetLinear(void* arkode_mem, int timedepend)

   Specifies that the implicit portion of the problem is linear.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *timedepend* -- flag denoting whether the Jacobian of
        :math:`f^I(t,y)` is time-dependent (1) or not (0).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Tightens the linear solver tolerances and takes only a
      single Newton iteration.  Calls :c:func:`ARKStepSetDeltaGammaMax()`
      to enforce Jacobian recomputation when the step size ratio changes
      by more than 100 times the unit roundoff (since nonlinear
      convergence is not tested).  Only applicable when used in
      combination with the modified or inexact Newton iteration (not the
      fixed-point solver).

      When :math:`f^I(t,y)` is time-dependent, all linear solver structures
      (Jacobian, preconditioner) will be updated preceding *each* implicit
      stage.  Thus one must balance the relative costs of such recomputation
      against the benefits of requiring only a single Newton linear solve.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetLinear` instead.


.. c:function:: int ARKStepSetNonlinear(void* arkode_mem)

   Specifies that the implicit portion of the problem is nonlinear.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This is the default behavior of ARKStep, so the function
      is primarily useful to undo a previous call to
      :c:func:`ARKStepSetLinear()`.  Calls
      :c:func:`ARKStepSetDeltaGammaMax()` to reset the step size ratio
      threshold to the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetNonlinear` instead.


.. c:function:: int ARKStepSetPredictorMethod(void* arkode_mem, int method)

   Specifies the method from :numref:`ARKODE.Mathematics.Predictors` to use
   for predicting implicit solutions.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *method* -- method choice (0 :math:`\le` *method* :math:`\le` 4):

        * 0 is the trivial predictor,

        * 1 is the maximum order (dense output) predictor,

        * 2 is the variable order predictor, that decreases the
          polynomial degree for more distant RK stages,

        * 3 is the cutoff order predictor, that uses the maximum order
          for early RK stages, and a first-order predictor for distant
          RK stages,

        * 4 is the bootstrap predictor, that uses a second-order
          predictor based on only information within the current step.
          **deprecated**

        * 5 is the minimum correction predictor, that uses all
          preceding stage information within the current step for
          prediction.
          **deprecated**

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The default value is 0.  If *method* is set to an
      undefined value, this default predictor will be used.

      Options 4 and 5 are currently not supported when solving a problem involving
      a non-identity mass matrix.  In that case, selection of *method* as 4 or 5 will
      instead default to the trivial predictor (*method* 0).  **Both of these options
      have been deprecated, and will be removed from a future release.**

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetPredictorMethod` instead.


.. c:function:: int ARKStepSetStagePredictFn(void* arkode_mem, ARKStagePredictFn PredictStage)

   Sets the user-supplied function to update the implicit stage predictor prior to
   execution of the nonlinear or linear solver algorithms that compute the implicit stage solution.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *PredictStage* -- name of user-supplied predictor function.  If ``NULL``, then any
        previously-provided stage prediction function will be disabled.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``

   **Notes:**
      See :numref:`ARKODE.Usage.StagePredictFn` for more information on
      this user-supplied routine.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetStagePredictFn` instead.


.. c:function:: int ARKStepSetNlsRhsFn(void* arkode_mem, ARKRhsFn nls_fi)

   Specifies an alternative implicit right-hand side function for evaluating
   :math:`f^I(t,y)` within nonlinear system function evaluations
   :eq:`ARKODE_Residual_MeqI` - :eq:`ARKODE_Residual_MTimeDep`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nls_fi* -- the alternative C function for computing the right-hand side
        function :math:`f^I(t,y)` in the ODE.

   **Return value:**
      * *ARK_SUCCESS* if successful.
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``.

   **Notes:**
      The default is to use the implicit right-hand side function
      provided to :c:func:`ARKStepCreate` in nonlinear system functions. If the
      input implicit right-hand side function is ``NULL``, the default is used.

      When using a non-default nonlinear solver, this function must be called
      *after* :c:func:`ARKStepSetNonlinearSolver()`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetNlsRhsFn` instead.


.. c:function:: int ARKStepSetMaxNonlinIters(void* arkode_mem, int maxcor)

   Specifies the maximum number of nonlinear solver
   iterations permitted per implicit stage solve within each time step.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *maxcor* -- maximum allowed solver iterations per stage :math:`(>0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value or if the SUNNONLINSOL module is ``NULL``
      * *ARK_NLS_OP_ERR* if the SUNNONLINSOL object returned a failure flag

   **Notes:**
      The default value is 3; set *maxcor* :math:`\le 0`
      to specify this default.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxNonlinIters` instead.


.. c:function:: int ARKStepSetNonlinConvCoef(void* arkode_mem, sunrealtype nlscoef)

   Specifies the safety factor :math:`\epsilon` used within the nonlinear
   solver convergence test :eq:`ARKODE_NonlinearTolerance`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nlscoef* -- coefficient in nonlinear solver convergence test :math:`(>0.0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The default value is 0.1; set *nlscoef* :math:`\le 0`
      to specify this default.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetNonlinConvCoef` instead.


.. c:function:: int ARKStepSetNonlinCRDown(void* arkode_mem, sunrealtype crdown)

   Specifies the constant :math:`c_r` used in estimating the nonlinear solver convergence rate :eq:`ARKODE_NonlinearCRate`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *crdown* -- nonlinear convergence rate estimation constant (default is 0.3).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any non-positive parameter will imply a reset to the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetNonlinCRDown` instead.


.. c:function:: int ARKStepSetNonlinRDiv(void* arkode_mem, sunrealtype rdiv)

   Specifies the nonlinear correction threshold :math:`r_{div}` from
   :eq:`ARKODE_NonlinearDivergence`, beyond which the iteration will be declared divergent.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *rdiv* -- tolerance on nonlinear correction size ratio to
        declare divergence (default is 2.3).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any non-positive parameter will imply a reset to the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetNonlinRDiv` instead.


.. c:function:: int ARKStepSetMaxConvFails(void* arkode_mem, int maxncf)

   Specifies the maximum number of nonlinear solver convergence
   failures permitted during one step, :math:`max_{ncf}` from
   :numref:`ARKODE.Mathematics.Error.Nonlinear`, before ARKStep will return with
   an error.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *maxncf* -- maximum allowed nonlinear solver convergence failures
        per step :math:`(>0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The default value is 10; set *maxncf* :math:`\le 0`
      to specify this default.

      Upon each convergence failure, ARKStep will first call the Jacobian
      setup routine and try again (if a Newton method is used).  If a
      convergence failure still occurs, the time step size is reduced by
      the factor *etacf* (set within :c:func:`ARKStepSetMaxCFailGrowth()`).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMaxConvFails` instead.


.. c:function:: int ARKStepSetDeduceImplicitRhs(void *arkode_mem, sunbooleantype deduce)

   Specifies if implicit stage derivatives are deduced without evaluating
   :math:`f^I`. See :numref:`ARKODE.Mathematics.Nonlinear` for more details.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *deduce* -- If ``SUNFALSE`` (default), the stage derivative is obtained
        by evaluating :math:`f^I` with the stage solution returned from the
        nonlinear solver. If ``SUNTRUE``, the stage derivative is deduced
        without an additional evaluation of :math:`f^I`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``

   .. versionadded:: 5.2.0

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetDeduceImplicitRhs` instead.


.. _ARKODE.Usage.ARKStep.ARKLsInputs:


Linear solver interface optional input functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. _ARKODE.Usage.ARKStep.ARKLsInputs.General:


Optional inputs for the ARKLS linear solver interface
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


.. c:function:: int ARKStepSetDeltaGammaMax(void* arkode_mem, sunrealtype dgmax)

   Specifies a scaled step size ratio tolerance, :math:`\Delta\gamma_{max}` from
   :numref:`ARKODE.Mathematics.Linear.Setup`, beyond which the linear solver
   setup routine will be signaled.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *dgmax* -- tolerance on step size ratio change before calling
        linear solver setup routine (default is 0.2).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      Any non-positive parameter will imply a reset to the default value.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetDeltaGammaMax` instead.



.. c:function:: int ARKStepSetLSetupFrequency(void* arkode_mem, int msbp)

   Specifies the frequency of calls to the linear solver setup
   routine, :math:`msbp` from :numref:`ARKODE.Mathematics.Linear.Setup`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *msbp* -- the linear solver setup frequency.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``

   **Notes:**
      Positive values of **msbp** specify the linear solver setup frequency. For
      example, an input of 1 means the setup function will be called every time
      step while an input of 2 means it will be called called every other time
      step. If **msbp** is 0, the default value of 20 will be used. A negative
      value forces a linear solver step at each implicit stage.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetLSetupFrequency` instead.


.. c:function:: int ARKStepSetJacEvalFrequency(void* arkode_mem, long int msbj)

   Specifies the number of steps after which the Jacobian information is
   considered out-of-date, :math:`msbj` from :numref:`ARKODE.Mathematics.Linear.Setup`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *msbj* -- the Jacobian re-computation or preconditioner update frequency.

   **Return value:**
      * *ARKLS_SUCCESS* if successful.
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``.
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``.

   **Notes:**
      If ``nstlj`` is the step number at which the Jacobian information was
      lasted updated and ``nst`` is the current step number,
      ``nst - nstlj >= msbj`` indicates that the Jacobian information will be updated
      during the next linear solver setup call.

      As the Jacobian update frequency is only checked *within* calls to the
      linear solver setup routine, Jacobian information may be more than
      ``msbj`` steps old when updated depending on when a linear solver setup
      call occurs. See :numref:`ARKODE.Mathematics.Linear.Setup`
      for more information on when linear solver setups are performed.

      Passing a value *msbj* :math:`\le 0` indicates to use the
      default value of 51.

      This function must be called *after* the ARKLS system solver interface has
      been initialized through a call to :c:func:`ARKStepSetLinearSolver()`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetJacEvalFrequency` instead.





.. _ARKODE.Usage.ARKStep.ARKLsInputs.MatrixBased:

Optional inputs for matrix-based ``SUNLinearSolver`` modules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


.. c:function:: int ARKStepSetJacFn(void* arkode_mem, ARKLsJacFn jac)

   Specifies the Jacobian approximation routine to
   be used for the matrix-based solver with the ARKLS interface.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *jac* -- name of user-supplied Jacobian approximation function.

   **Return value:**
      * *ARKLS_SUCCESS*  if successful
      * *ARKLS_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Notes:**
      This routine must be called after the ARKLS linear
      solver interface has been initialized through a call to
      :c:func:`ARKStepSetLinearSolver()`.

      By default, ARKLS uses an internal difference quotient function for
      the :ref:`SUNMATRIX_DENSE <SUNMatrix.Dense>` and
      :ref:`SUNMATRIX_BAND <SUNMatrix.Band>` modules.  If ``NULL`` is passed
      in for *jac*, this default is used. An error will occur if no *jac* is
      supplied when using other matrix types.

      The function type :c:func:`ARKLsJacFn()` is described in
      :numref:`ARKODE.Usage.UserSupplied`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetJacFn` instead.


.. c:function:: int ARKStepSetLinSysFn(void* arkode_mem, ARKLsLinSysFn linsys)

   Specifies the linear system approximation routine to be used for the
   matrix-based solver with the ARKLS interface.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *linsys* -- name of user-supplied linear system approximation function.

   **Return value:**
      * *ARKLS_SUCCESS*  if successful
      * *ARKLS_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Notes:**
      This routine must be called after the ARKLS linear
      solver interface has been initialized through a call to
      :c:func:`ARKStepSetLinearSolver()`.

      By default, ARKLS uses an internal linear system function that leverages the
      SUNMATRIX API to form the system :math:`M - \gamma J`.  If ``NULL`` is passed
      in for *linsys*, this default is used.

      The function type :c:func:`ARKLsLinSysFn()` is described in
      :numref:`ARKODE.Usage.UserSupplied`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetLinSysFn` instead.


.. c:function:: int ARKStepSetMassFn(void* arkode_mem, ARKLsMassFn mass)

   Specifies the mass matrix approximation routine to be used for the
   matrix-based solver with the ARKLS interface.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *mass* -- name of user-supplied mass matrix approximation function.

   **Return value:**
      * *ARKLS_SUCCESS*  if successful
      * *ARKLS_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARKLS_MASSMEM_NULL* if the mass matrix solver memory was ``NULL``
      * *ARKLS_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      This routine must be called after the ARKLS mass matrix
      solver interface has been initialized through a call to
      :c:func:`ARKStepSetMassLinearSolver()`.

      Since there is no default difference quotient function for mass
      matrices, *mass* must be non-``NULL``.

      The function type :c:func:`ARKLsMassFn()` is described in
      :numref:`ARKODE.Usage.UserSupplied`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMassFn` instead.


.. c:function:: int ARKStepSetLinearSolutionScaling(void* arkode_mem, sunbooleantype onoff)

   Enables or disables scaling the linear system solution to account for a
   change in :math:`\gamma` in the linear system. For more details see
   :numref:`SUNLinSol.Lagged_matrix`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *onoff* -- flag to enable (``SUNTRUE``) or disable (``SUNFALSE``)
        scaling

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_ILL_INPUT* if the attached linear solver is not matrix-based

   **Notes:**
      Linear solution scaling is enabled by default when a matrix-based
      linear solver is attached.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetLinearSolutionScaling` instead.


.. _ARKODE.Usage.ARKStep.ARKLsInputs.MatrixFree:

Optional inputs for matrix-free ``SUNLinearSolver`` modules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


.. c:function:: int ARKStepSetJacTimes(void* arkode_mem, ARKLsJacTimesSetupFn jtsetup, ARKLsJacTimesVecFn jtimes)

   Specifies the Jacobian-times-vector setup and product functions.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *jtsetup* -- user-defined Jacobian-vector setup function.
        Pass ``NULL`` if no setup is necessary.
      * *jtimes* -- user-defined Jacobian-vector product function.

   **Return value:**
      * *ARKLS_SUCCESS* if successful.
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``.
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``.
      * *ARKLS_ILL_INPUT* if an input has an illegal value.
      * *ARKLS_SUNLS_FAIL* if an error occurred when setting up
        the Jacobian-vector product in the ``SUNLinearSolver``
        object used by the ARKLS interface.

   **Notes:**
      The default is to use an internal finite difference
      quotient for *jtimes* and to leave out *jtsetup*.  If ``NULL`` is
      passed to *jtimes*, these defaults are used.  A user may
      specify non-``NULL`` *jtimes* and ``NULL`` *jtsetup* inputs.

      This function must be called *after* the ARKLS system solver
      interface has been initialized through a call to
      :c:func:`ARKStepSetLinearSolver()`.

      The function types :c:type:`ARKLsJacTimesSetupFn` and
      :c:type:`ARKLsJacTimesVecFn` are described in
      :numref:`ARKODE.Usage.UserSupplied`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetJacTimes` instead.



.. c:function:: int ARKStepSetJacTimesRhsFn(void* arkode_mem, ARKRhsFn jtimesRhsFn)

   Specifies an alternative implicit right-hand side function for use in the
   internal Jacobian-vector product difference quotient approximation.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *jtimesRhsFn* -- the name of the C function (of type
        :c:func:`ARKRhsFn()`) defining the alternative right-hand side function.

   **Return value:**
      * *ARKLS_SUCCESS* if successful.
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``.
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``.
      * *ARKLS_ILL_INPUT* if an input has an illegal value.

   **Notes:**
      The default is to use the implicit right-hand side function
      provided to :c:func:`ARKStepCreate` in the internal difference quotient. If
      the input implicit right-hand side function is ``NULL``, the default is used.

      This function must be called *after* the ARKLS system solver interface has
      been initialized through a call to :c:func:`ARKStepSetLinearSolver()`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetJacTimesRhsFn` instead.


.. c:function:: int ARKStepSetMassTimes(void* arkode_mem, ARKLsMassTimesSetupFn mtsetup, ARKLsMassTimesVecFn mtimes, void* mtimes_data)

   Specifies the mass matrix-times-vector setup and product functions.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *mtsetup* -- user-defined mass matrix-vector setup function.
        Pass ``NULL`` if no setup is necessary.
      * *mtimes* -- user-defined mass matrix-vector product function.
      * *mtimes_data* -- a pointer to user data, that will be supplied
        to both the *mtsetup* and *mtimes* functions.

   **Return value:**
      * *ARKLS_SUCCESS* if successful.
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``.
      * *ARKLS_MASSMEM_NULL* if the mass matrix solver memory was ``NULL``.
      * *ARKLS_ILL_INPUT* if an input has an illegal value.
      * *ARKLS_SUNLS_FAIL* if an error occurred when setting up
        the mass-matrix-vector product in the ``SUNLinearSolver``
        object used by the ARKLS interface.

   **Notes:**
      There is no default finite difference quotient for
      *mtimes*, so if using the ARKLS mass matrix solver interface with
      NULL-valued SUNMATRIX input :math:`M`, and this routine is called
      with NULL-valued *mtimes*, an error will occur.  A user may
      specify ``NULL`` for *mtsetup*.

      This function must be called *after* the ARKLS mass
      matrix solver interface has been initialized through a call to
      :c:func:`ARKStepSetMassLinearSolver()`.

      The function types :c:type:`ARKLsMassTimesSetupFn` and
      :c:type:`ARKLsMassTimesVecFn` are described in
      :numref:`ARKODE.Usage.UserSupplied`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMassTimes` instead.



.. _ARKODE.Usage.ARKStep.ARKLsInputs.Iterative:

Optional inputs for iterative ``SUNLinearSolver`` modules
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""


.. c:function:: int ARKStepSetPreconditioner(void* arkode_mem, ARKLsPrecSetupFn psetup, ARKLsPrecSolveFn psolve)

   Specifies the user-supplied preconditioner setup and solve functions.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *psetup* -- user defined preconditioner setup function.  Pass
        ``NULL`` if no setup is needed.
      * *psolve* -- user-defined preconditioner solve function.

   **Return value:**
      * *ARKLS_SUCCESS* if successful.
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``.
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``.
      * *ARKLS_ILL_INPUT* if an input has an illegal value.
      * *ARKLS_SUNLS_FAIL* if an error occurred when setting up
        preconditioning in the ``SUNLinearSolver`` object used
        by the ARKLS interface.

   **Notes:**
      The default is ``NULL`` for both arguments (i.e., no
      preconditioning).

      This function must be called *after* the ARKLS system solver
      interface has been initialized through a call to
      :c:func:`ARKStepSetLinearSolver()`.

      Both of the function types :c:func:`ARKLsPrecSetupFn()` and
      :c:func:`ARKLsPrecSolveFn()` are described in
      :numref:`ARKODE.Usage.UserSupplied`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetPreconditioner` instead.


.. c:function:: int ARKStepSetMassPreconditioner(void* arkode_mem, ARKLsMassPrecSetupFn psetup, ARKLsMassPrecSolveFn psolve)

   Specifies the mass matrix preconditioner setup and solve functions.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *psetup* -- user defined preconditioner setup function.  Pass
        ``NULL`` if no setup is to be done.
      * *psolve* -- user-defined preconditioner solve function.

   **Return value:**
      * *ARKLS_SUCCESS* if successful.
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``.
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``.
      * *ARKLS_ILL_INPUT* if an input has an illegal value.
      * *ARKLS_SUNLS_FAIL* if an error occurred when setting up
        preconditioning in the ``SUNLinearSolver`` object used
        by the ARKLS interface.

   **Notes:**
      This function must be called *after* the ARKLS mass
      matrix solver interface has been initialized through a call to
      :c:func:`ARKStepSetMassLinearSolver()`.

      The default is ``NULL`` for both arguments (i.e. no
      preconditioning).

      Both of the function types :c:func:`ARKLsMassPrecSetupFn()` and
      :c:func:`ARKLsMassPrecSolveFn()` are described in
      :numref:`ARKODE.Usage.UserSupplied`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMassPreconditioner` instead.


.. c:function:: int ARKStepSetEpsLin(void* arkode_mem, sunrealtype eplifac)

   Specifies the factor :math:`\epsilon_L` by which the tolerance on
   the nonlinear iteration is multiplied to get a tolerance on the
   linear iteration.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *eplifac* -- linear convergence safety factor.

   **Return value:**
      * *ARKLS_SUCCESS* if successful.
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``.
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``.
      * *ARKLS_ILL_INPUT* if an input has an illegal value.

   **Notes:**
      Passing a value *eplifac* :math:`\le 0` indicates to use the
      default value of 0.05.

      This function must be called *after* the ARKLS system solver
      interface has been initialized through a call to
      :c:func:`ARKStepSetLinearSolver()`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetEpsLin` instead.


.. c:function:: int ARKStepSetMassEpsLin(void* arkode_mem, sunrealtype eplifac)

   Specifies the factor by which the tolerance on the nonlinear
   iteration is multiplied to get a tolerance on the mass matrix
   linear iteration.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *eplifac* -- linear convergence safety factor.

   **Return value:**
      * *ARKLS_SUCCESS* if successful.
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``.
      * *ARKLS_MASSMEM_NULL* if the mass matrix solver memory was ``NULL``.
      * *ARKLS_ILL_INPUT* if an input has an illegal value.

   **Notes:**
      This function must be called *after* the ARKLS mass
      matrix solver interface has been initialized through a call to
      :c:func:`ARKStepSetMassLinearSolver()`.

      Passing a value *eplifac* :math:`\le 0` indicates to use the default value
      of 0.05.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMassEpsLin` instead.


.. c:function:: int ARKStepSetLSNormFactor(void* arkode_mem, sunrealtype nrmfac)

   Specifies the factor to use when converting from the integrator tolerance
   (WRMS norm) to the linear solver tolerance (L2 norm) for Newton linear system
   solves.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nrmfac* -- the norm conversion factor. If *nrmfac* is:

        :math:`> 0` then the provided value is used.

        :math:`= 0` then the conversion factor is computed using the vector
        length i.e., ``nrmfac = sqrt(N_VGetLength(y))`` (*default*).

        :math:`< 0` then the conversion factor is computed using the vector dot
        product i.e., ``nrmfac = sqrt(N_VDotProd(v,v))`` where all the entries
        of ``v`` are one.

   **Return value:**
      * *ARK_SUCCESS* if successful.
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``.

   **Notes:**
      This function must be called *after* the ARKLS system solver interface has
      been initialized through a call to :c:func:`ARKStepSetLinearSolver()`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetLSNormFactor` instead.


.. c:function:: int ARKStepSetMassLSNormFactor(void* arkode_mem, sunrealtype nrmfac)

   Specifies the factor to use when converting from the integrator tolerance
   (WRMS norm) to the linear solver tolerance (L2 norm) for mass matrix linear
   system solves.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nrmfac* -- the norm conversion factor. If *nrmfac* is:

        :math:`> 0` then the provided value is used.

        :math:`= 0` then the conversion factor is computed using the vector
        length i.e., ``nrmfac = sqrt(N_VGetLength(y))`` (*default*).

        :math:`< 0` then the conversion factor is computed using the vector dot
        product i.e., ``nrmfac = sqrt(N_VDotProd(v,v))`` where all the entries
        of ``v`` are one.

   **Return value:**
      * *ARK_SUCCESS* if successful.
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``.

   **Notes:**
      This function must be called *after* the ARKLS mass matrix solver interface
      has been initialized through a call to :c:func:`ARKStepSetMassLinearSolver()`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetMassLSNormFactor` instead.



Rootfinding optional input functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. c:function:: int ARKStepSetRootDirection(void* arkode_mem, int* rootdir)

   Specifies the direction of zero-crossings to be located and returned.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *rootdir* -- state array of length *nrtfn*, the number of root
        functions :math:`g_i` (the value of *nrtfn* was supplied in
        the call to :c:func:`ARKStepRootInit()`).  If ``rootdir[i] ==
        0`` then crossing in either direction for :math:`g_i` should be
        reported.  A value of +1 or -1 indicates that the solver
        should report only zero-crossings where :math:`g_i` is
        increasing or decreasing, respectively.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``
      * *ARK_ILL_INPUT* if an argument had an illegal value

   **Notes:**
      The default behavior is to monitor for both zero-crossing directions.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetRootDirection` instead.


.. c:function:: int ARKStepSetNoInactiveRootWarn(void* arkode_mem)

   Disables issuing a warning if some root function appears
   to be identically zero at the beginning of the integration.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``

   **Notes:**
      ARKStep will not report the initial conditions as a
      possible zero-crossing (assuming that one or more components
      :math:`g_i` are zero at the initial time).  However, if it appears
      that some :math:`g_i` is identically zero at the initial time
      (i.e., :math:`g_i` is zero at the initial time *and* after the
      first step), ARKStep will issue a warning which can be disabled with
      this optional input function.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeSetNoInactiveRootWarn` instead.



.. _ARKODE.Usage.ARKStep.InterpolatedOutput:

Interpolated output function
--------------------------------


.. c:function:: int ARKStepGetDky(void* arkode_mem, sunrealtype t, int k, N_Vector dky)

   Computes the *k*-th derivative of the function
   :math:`y` at the time *t*,
   i.e. :math:`y^{(k)}(t)`, for values of the
   independent variable satisfying :math:`t_n-h_n \le t \le t_n`, with
   :math:`t_n` as current internal time reached, and :math:`h_n` is
   the last internal step size successfully used by the solver.  This
   routine uses an interpolating polynomial of degree *min(degree, 5)*,
   where *degree* is the argument provided to
   :c:func:`ARKStepSetInterpolantDegree()`.  The user may request *k* in the
   range {0,..., *min(degree, kmax)*} where *kmax* depends on the choice of
   interpolation module. For Hermite interpolants *kmax = 5* and for Lagrange
   interpolants *kmax = 3*.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *t* -- the value of the independent variable at which the
        derivative is to be evaluated.
      * *k* -- the derivative order requested.
      * *dky* -- output vector (must be allocated by the user).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_BAD_K* if *k* is not in the range {0,..., *min(degree, kmax)*}.
      * *ARK_BAD_T* if *t* is not in the interval :math:`[t_n-h_n, t_n]`
      * *ARK_BAD_DKY* if the *dky* vector was ``NULL``
      * *ARK_MEM_NULL* if the ARKStep memory is ``NULL``

   **Notes:**
      It is only legal to call this function after a successful
      return from :c:func:`ARKStepEvolve()`.

      A user may access the values :math:`t_n` and :math:`h_n` via the
      functions :c:func:`ARKStepGetCurrentTime()` and
      :c:func:`ARKStepGetLastStep()`, respectively.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetDky` instead.



.. _ARKODE.Usage.ARKStep.OptionalOutputs:

Optional output functions
------------------------------

.. _ARKODE.Usage.ARKStep.ARKStepMainOutputs:

Main solver optional output functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. c:function:: int ARKStepGetWorkSpace(void* arkode_mem, long int* lenrw, long int* leniw)

   Returns the ARKStep real and integer workspace sizes.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *lenrw* -- the number of ``sunrealtype`` values in the ARKStep workspace.
      * *leniw* -- the number of integer values in the ARKStep workspace.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetWorkSpace` instead.


.. c:function:: int ARKStepGetNumSteps(void* arkode_mem, long int* nsteps)

   Returns the cumulative number of internal steps taken by
   the solver (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nsteps* -- number of steps taken in the solver.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumSteps` instead.


.. c:function:: int ARKStepGetActualInitStep(void* arkode_mem, sunrealtype* hinused)

   Returns the value of the integration step size used on the first step.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *hinused* -- actual value of initial step size.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      Even if the value of the initial integration step was
      specified by the user through a call to
      :c:func:`ARKStepSetInitStep()`, this value may have been changed by
      ARKStep to ensure that the step size fell within the prescribed
      bounds :math:`(h_{min} \le h_0 \le h_{max})`, or to satisfy the
      local error test condition, or to ensure convergence of the
      nonlinear solver.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetActualInitStep` instead.


.. c:function:: int ARKStepGetLastStep(void* arkode_mem, sunrealtype* hlast)

   Returns the integration step size taken on the last successful
   internal step.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *hlast* -- step size taken on the last internal step.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetLastStep` instead.


.. c:function:: int ARKStepGetCurrentStep(void* arkode_mem, sunrealtype* hcur)

   Returns the integration step size to be attempted on the next internal step.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *hcur* -- step size to be attempted on the next internal step.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetCurrentStep` instead.


.. c:function:: int ARKStepGetCurrentTime(void* arkode_mem, sunrealtype* tcur)

   Returns the current internal time reached by the solver.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *tcur* -- current internal time reached.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetCurrentTime` instead.


.. c:function:: int ARKStepGetCurrentState(void *arkode_mem, N_Vector *ycur)

   Returns the current internal solution reached by the solver.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *ycur* -- current internal solution.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      Users should exercise extreme caution when using this function,
      as altering values of *ycur* may lead to undesirable behavior, depending
      on the particular use case and on when this routine is called.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetCurrentState` instead.


.. c:function:: int ARKStepGetCurrentGamma(void *arkode_mem, sunrealtype *gamma)

   Returns the current internal value of :math:`\gamma` used in the implicit
   solver Newton matrix (see equation :eq:`ARKODE_NewtonMatrix`).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *gamma* -- current step size scaling factor in the Newton system.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetCurrentGamma` instead.


.. c:function:: int ARKStepGetTolScaleFactor(void* arkode_mem, sunrealtype* tolsfac)

   Returns a suggested factor by which the user's
   tolerances should be scaled when too much accuracy has been
   requested for some internal step.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *tolsfac* -- suggested scaling factor for user-supplied tolerances.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetTolScaleFactor` instead.


.. c:function:: int ARKStepGetErrWeights(void* arkode_mem, N_Vector eweight)

   Returns the current error weight vector.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *eweight* -- solution error weights at the current time.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      The user must allocate space for *eweight*, that will be
      filled in by this function.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetErrWeights` instead.


.. c:function:: int ARKStepGetResWeights(void* arkode_mem, N_Vector rweight)

   Returns the current residual weight vector.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *rweight* -- residual error weights at the current time.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      The user must allocate space for *rweight*, that will be
      filled in by this function.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetResWeights` instead.


.. c:function:: int ARKStepGetStepStats(void* arkode_mem, long int* nsteps, sunrealtype* hinused, sunrealtype* hlast, sunrealtype* hcur, sunrealtype* tcur)

   Returns many of the most useful optional outputs in a single call.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nsteps* -- number of steps taken in the solver.
      * *hinused* -- actual value of initial step size.
      * *hlast* -- step size taken on the last internal step.
      * *hcur* -- step size to be attempted on the next internal step.
      * *tcur* -- current internal time reached.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetStepStats` instead.


.. c:function:: int ARKStepPrintAllStats(void* arkode_mem, FILE* outfile, SUNOutputFormat fmt)

   Outputs all of the integrator, nonlinear solver, linear solver, and other
   statistics.

   **Arguments:**
     * *arkode_mem* -- pointer to the ARKStep memory block.
     * *outfile* -- pointer to output file.
     * *fmt* -- the output format:

       * :c:enumerator:`SUN_OUTPUTFORMAT_TABLE` -- prints a table of values
       * :c:enumerator:`SUN_OUTPUTFORMAT_CSV` -- prints a comma-separated list
         of key and value pairs e.g., ``key1,value1,key2,value2,...``

   **Return value:**
     * *ARK_SUCCESS* -- if the output was successfully.
     * *ARK_MEM_NULL* -- if the ARKStep memory was ``NULL``.
     * *ARK_ILL_INPUT* -- if an invalid formatting option was provided.

   .. note::

      The Python module ``tools/suntools`` provides utilities to read and output
      the data from a SUNDIALS CSV output file using the key and value pair
      format.

   .. versionadded:: 5.2.0

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodePrintAllStats` instead.


.. c:function:: char* ARKStepGetReturnFlagName(long int flag)

   Returns the name of the ARKStep constant corresponding to *flag*.
   See :ref:`ARKODE.Constants`.

   **Arguments:**
      * *flag* -- a return flag from an ARKStep function.

   **Return value:**
   The return value is a string containing the name of
   the corresponding constant.

   .. warning::

      The user is responsible for freeing the returned string.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetReturnFlagName` instead.



.. c:function:: int ARKStepGetNumExpSteps(void* arkode_mem, long int* expsteps)

   Returns the cumulative number of stability-limited steps
   taken by the solver (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *expsteps* -- number of stability-limited steps taken in the solver.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumExpSteps` instead.


.. c:function:: int ARKStepGetNumAccSteps(void* arkode_mem, long int* accsteps)

   Returns the cumulative number of accuracy-limited steps
   taken by the solver (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *accsteps* -- number of accuracy-limited steps taken in the solver.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumAccSteps` instead.


.. c:function:: int ARKStepGetNumStepAttempts(void* arkode_mem, long int* step_attempts)

   Returns the cumulative number of steps attempted by the solver (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *step_attempts* -- number of steps attempted by solver.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumStepAttempts` instead.


.. c:function:: int ARKStepGetNumRhsEvals(void* arkode_mem, long int* nfe_evals, long int* nfi_evals)

   Returns the number of calls to the user's right-hand
   side functions, :math:`f^E` and :math:`f^I` (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nfe_evals* -- number of calls to the user's :math:`f^E(t,y)` function.
      * *nfi_evals* -- number of calls to the user's :math:`f^I(t,y)` function.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      The *nfi_evals* value does not account for calls made to
      :math:`f^I` by a linear solver or preconditioner module.

   .. deprecated:: 6.2.0

      Use :c:func:`ARKodeGetNumRhsEvals` instead.


.. c:function:: int ARKStepGetNumErrTestFails(void* arkode_mem, long int* netfails)

   Returns the number of local error test failures that
   have occurred (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *netfails* -- number of error test failures.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumErrTestFails` instead.


.. c:function:: int ARKStepGetNumStepSolveFails(void* arkode_mem, long int* ncnf)

   Returns the number of failed steps due to a nonlinear solver failure (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *ncnf* -- number of step failures.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumStepSolveFails` instead.


.. c:function:: int ARKStepGetCurrentButcherTables(void* arkode_mem, ARKodeButcherTable *Bi, ARKodeButcherTable *Be)

   Returns the explicit and implicit Butcher tables
   currently in use by the solver.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *Bi* -- pointer to the implicit Butcher table structure.
      * *Be* -- pointer to the explicit Butcher table structure.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Note:**  The :c:type:`ARKodeButcherTable` data structure is defined as a
   pointer to the following C structure:

   .. code-block:: c

      typedef struct ARKStepButcherTableMem {

        int q;           /* method order of accuracy       */
        int p;           /* embedding order of accuracy    */
        int stages;      /* number of stages               */
        sunrealtype **A;    /* Butcher table coefficients     */
        sunrealtype *c;     /* canopy node coefficients       */
        sunrealtype *b;     /* root node coefficients         */
        sunrealtype *d;     /* embedding coefficients         */

      } *ARKStepButcherTable;

   For more details see :numref:`ARKodeButcherTable`.


.. c:function:: int ARKStepGetEstLocalErrors(void* arkode_mem, N_Vector ele)

   Returns the vector of estimated local truncation errors
   for the current step.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *ele* -- vector of estimated local truncation errors.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      The user must allocate space for *ele*, that will be
      filled in by this function.

      The values returned in *ele* are valid only after a successful call
      to :c:func:`ARKStepEvolve()` (i.e., it returned a non-negative value).

      The *ele* vector, together with the *eweight* vector from
      :c:func:`ARKStepGetErrWeights()`, can be used to determine how the
      various components of the system contributed to the estimated local
      error test.  Specifically, that error test uses the WRMS norm of a
      vector whose components are the products of the components of these
      two vectors.  Thus, for example, if there were recent error test
      failures, the components causing the failures are those with largest
      values for the products, denoted loosely as ``eweight[i]*ele[i]``.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetEstLocalErrors` instead.


.. c:function:: int ARKStepGetTimestepperStats(void* arkode_mem, long int* expsteps, long int* accsteps, long int* step_attempts, long int* nfe_evals, long int* nfi_evals, long int* nlinsetups, long int* netfails)

   Returns many of the most useful time-stepper statistics in a single call.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *expsteps* -- number of stability-limited steps taken in the solver.
      * *accsteps* -- number of accuracy-limited steps taken in the solver.
      * *step_attempts* -- number of steps attempted by the solver.
      * *nfe_evals* -- number of calls to the user's :math:`f^E(t,y)` function.
      * *nfi_evals* -- number of calls to the user's :math:`f^I(t,y)` function.
      * *nlinsetups* -- number of linear solver setup calls made.
      * *netfails* -- number of error test failures.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``



.. c:function:: int ARKStepGetNumConstrFails(void* arkode_mem, long int* nconstrfails)

   Returns the cumulative number of constraint test failures (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nconstrfails* -- number of constraint test failures.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumConstrFails` instead.


.. c:function:: int ARKStepGetUserData(void* arkode_mem, void** user_data)

   Returns the user data pointer previously set with
   :c:func:`ARKStepSetUserData`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *user_data* -- memory reference to a user data pointer

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. versionadded:: 5.3.0

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetUserData` instead.


.. _ARKODE.Usage.ARKStep.ARKStepImplicitSolverOutputs:

Implicit solver optional output functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. c:function:: int ARKStepGetNumLinSolvSetups(void* arkode_mem, long int* nlinsetups)

   Returns the number of calls made to the linear solver's
   setup routine (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nlinsetups* -- number of linear solver setup calls made.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the nonlinear
   solver object; the counter is reset whenever a new nonlinear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumLinSolvSetups` instead.


.. c:function:: int ARKStepGetNumNonlinSolvIters(void* arkode_mem, long int* nniters)

   Returns the number of nonlinear solver iterations
   performed (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nniters* -- number of nonlinear iterations performed.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARK_NLS_OP_ERR* if the SUNNONLINSOL object returned a failure flag

   **Note:** This is only accumulated for the "life" of the nonlinear
   solver object; the counter is reset whenever a new nonlinear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumNonlinSolvIters` instead.


.. c:function:: int ARKStepGetNumNonlinSolvConvFails(void* arkode_mem, long int* nncfails)

   Returns the number of nonlinear solver convergence
   failures that have occurred (so far).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nncfails* -- number of nonlinear convergence failures.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the nonlinear
   solver object; the counter is reset whenever a new nonlinear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumNonlinSolvConvFails` instead.


.. c:function:: int ARKStepGetNonlinSolvStats(void* arkode_mem, long int* nniters, long int* nncfails)

   Returns all of the nonlinear solver statistics in a single call.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nniters* -- number of nonlinear iterations performed.
      * *nncfails* -- number of nonlinear convergence failures.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARK_NLS_OP_ERR* if the SUNNONLINSOL object returned a failure flag

   **Note:** This is only accumulated for the "life" of the nonlinear
   solver object; the counters are reset whenever a new nonlinear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNonlinSolvStats` instead.



.. _ARKODE.Usage.ARKStep.ARKStepRootOutputs:

Rootfinding optional output functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. c:function:: int ARKStepGetRootInfo(void* arkode_mem, int* rootsfound)

   Returns an array showing which functions were found to
   have a root.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *rootsfound* -- array of length *nrtfn* with the indices of the
        user functions :math:`g_i` found to have a root (the value of
        *nrtfn* was supplied in the call to
        :c:func:`ARKStepRootInit()`).  For :math:`i = 0 \ldots`
        *nrtfn*-1, ``rootsfound[i]`` is nonzero if :math:`g_i` has a
        root, and 0 if not.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      The user must allocate space for *rootsfound* prior to
      calling this function.

      For the components of :math:`g_i` for which a root was found, the
      sign of ``rootsfound[i]`` indicates the direction of
      zero-crossing.  A value of +1 indicates that :math:`g_i` is
      increasing, while a value of -1 indicates a decreasing :math:`g_i`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetRootInfo` instead.


.. c:function:: int ARKStepGetNumGEvals(void* arkode_mem, long int* ngevals)

   Returns the cumulative number of calls made to the
   user's root function :math:`g`.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *ngevals* -- number of calls made to :math:`g` so far.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumGEvals` instead.


.. _ARKODE.Usage.ARKStep.ARKLsOutputs:

Linear solver interface optional output functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. c:function:: int ARKStepGetJac(void* arkode_mem, SUNMatrix* J)

   Returns the internally stored copy of the Jacobian matrix of the ODE
   implicit right-hand side function.

   :param arkode_mem: the ARKStep memory structure
   :param J: the Jacobian matrix

   :retval ARKLS_SUCCESS: the output value has been successfully set
   :retval ARKLS_MEM_NULL: ``arkode_mem`` was ``NULL``
   :retval ARKLS_LMEM_NULL: the linear solver interface has not been initialized

   .. warning::

      This function is provided for debugging purposes and the values in the
      returned matrix should not be altered.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetJac` instead.


.. c:function:: int ARKStepGetJacTime(void* arkode_mem, sunrealtype* t_J)

   Returns the time at which the internally stored copy of the Jacobian matrix
   of the ODE implicit right-hand side function was evaluated.

   :param arkode_mem: the ARKStep memory structure
   :param t_J: the time at which the Jacobian was evaluated

   :retval ARKLS_SUCCESS: the output value has been successfully set
   :retval ARKLS_MEM_NULL: ``arkode_mem`` was ``NULL``
   :retval ARKLS_LMEM_NULL: the linear solver interface has not been initialized

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetJacTime` instead.


.. c:function:: int ARKStepGetJacNumSteps(void* arkode_mem, long int* nst_J)

   Returns the value of the internal step counter at which the internally stored copy of the
   Jacobian matrix of the ODE implicit right-hand side function was evaluated.

   :param arkode_mem: the ARKStep memory structure
   :param nst_J: the value of the internal step counter at which the Jacobian was evaluated

   :retval ARKLS_SUCCESS: the output value has been successfully set
   :retval ARKLS_MEM_NULL: ``arkode_mem`` was ``NULL``
   :retval ARKLS_LMEM_NULL: the linear solver interface has not been initialized

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetJacNumSteps` instead.


.. c:function:: int ARKStepGetLinWorkSpace(void* arkode_mem, long int* lenrwLS, long int* leniwLS)

   Returns the real and integer workspace used by the ARKLS linear solver interface.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *lenrwLS* -- the number of ``sunrealtype`` values in the ARKLS workspace.
      * *leniwLS* -- the number of integer values in the ARKLS workspace.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Notes:**
      The workspace requirements reported by this routine
      correspond only to memory allocated within this interface and to
      memory allocated by the ``SUNLinearSolver`` object attached
      to it.  The template Jacobian matrix allocated by the user outside
      of ARKLS is not included in this report.

      In a parallel setting, the above values are global (i.e. summed over all
      processors).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetLinWorkSpace` instead.


.. c:function:: int ARKStepGetNumJacEvals(void* arkode_mem, long int* njevals)

   Returns the number of Jacobian evaluations.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *njevals* -- number of Jacobian evaluations.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new linear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumJacEvals` instead.


.. c:function:: int ARKStepGetNumPrecEvals(void* arkode_mem, long int* npevals)

   Returns the total number of preconditioner evaluations,
   i.e. the number of calls made to *psetup* with ``jok`` = ``SUNFALSE`` and
   that returned ``*jcurPtr`` = ``SUNTRUE``.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *npevals* -- the current number of calls to *psetup*.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new linear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumPrecEvals` instead.


.. c:function:: int ARKStepGetNumPrecSolves(void* arkode_mem, long int* npsolves)

   Returns the number of calls made to the preconditioner
   solve function, *psolve*.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *npsolves* -- the number of calls to *psolve*.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new linear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumPrecSolves` instead.


.. c:function:: int ARKStepGetNumLinIters(void* arkode_mem, long int* nliters)

   Returns the cumulative number of linear iterations.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nliters* -- the current number of linear iterations.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new linear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumLinIters` instead.


.. c:function:: int ARKStepGetNumLinConvFails(void* arkode_mem, long int* nlcfails)

   Returns the cumulative number of linear convergence failures.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nlcfails* -- the current number of linear convergence failures.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new linear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumLinConvFails` instead.


.. c:function:: int ARKStepGetNumJTSetupEvals(void* arkode_mem, long int* njtsetup)

   Returns the cumulative number of calls made to the user-supplied
   Jacobian-vector setup function, *jtsetup*.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *njtsetup* -- the current number of calls to *jtsetup*.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new linear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumJTSetupEvals` instead.


.. c:function:: int ARKStepGetNumJtimesEvals(void* arkode_mem, long int* njvevals)

   Returns the cumulative number of calls made to the
   Jacobian-vector product function, *jtimes*.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *njvevals* -- the current number of calls to *jtimes*.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new linear solver
   module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumJtimesEvals` instead.


.. c:function:: int ARKStepGetNumLinRhsEvals(void* arkode_mem, long int* nfevalsLS)

   Returns the number of calls to the user-supplied implicit
   right-hand side function :math:`f^I` for finite difference
   Jacobian or Jacobian-vector product approximation.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nfevalsLS* -- the number of calls to the user implicit
        right-hand side function.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Notes:**
      The value *nfevalsLS* is incremented only if the default
      internal difference quotient function is used.

      This is only accumulated for the "life" of the linear
      solver object; the counter is reset whenever a new linear solver
      module is "attached" to ARKStep, or when ARKStep is resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumLinRhsEvals` instead.


.. c:function:: int ARKStepGetLastLinFlag(void* arkode_mem, long int* lsflag)

   Returns the last return value from an ARKLS routine.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *lsflag* -- the value of the last return flag from an
        ARKLS function.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Notes:**
      If the ARKLS setup function failed when using the
      ``SUNLINSOL_DENSE`` or ``SUNLINSOL_BAND`` modules, then the value
      of *lsflag* is equal to the column index (numbered from one) at
      which a zero diagonal element was encountered during the LU
      factorization of the (dense or banded) Jacobian matrix.  For all
      other failures, *lsflag* is negative.

      Otherwise, if the ARKLS setup function failed
      (:c:func:`ARKStepEvolve()` returned *ARK_LSETUP_FAIL*), then
      *lsflag* will be *SUNLS_PSET_FAIL_UNREC*, *SUNLS_ASET_FAIL_UNREC*
      or *SUN_ERR_EXT_FAIL*.

      If the ARKLS solve function failed (:c:func:`ARKStepEvolve()`
      returned *ARK_LSOLVE_FAIL*), then *lsflag* contains the error
      return flag from the ``SUNLinearSolver`` object, which will
      be one of:
      *SUN_ERR_ARG_CORRUPTRRUPT*, indicating that the ``SUNLinearSolver``
      memory is ``NULL``;
      *SUNLS_ATIMES_NULL*, indicating that a matrix-free iterative solver
      was provided, but is missing a routine for the matrix-vector product
      approximation,
      *SUNLS_ATIMES_FAIL_UNREC*, indicating an unrecoverable failure in
      the :math:`Jv` function;
      *SUNLS_PSOLVE_NULL*, indicating that an iterative linear solver was
      configured to use preconditioning, but no preconditioner solve
      routine was provided,
      *SUNLS_PSOLVE_FAIL_UNREC*, indicating that the preconditioner solve
      function failed unrecoverably;
      *SUNLS_GS_FAIL*, indicating a failure in the Gram-Schmidt procedure
      (SPGMR and SPFGMR only);
      *SUNLS_QRSOL_FAIL*, indicating that the matrix :math:`R` was found
      to be singular during the QR solve phase (SPGMR and SPFGMR only); or
      *SUN_ERR_EXT_FAIL*, indicating an unrecoverable failure in
      an external iterative linear solver package.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetLastLinFlag` instead.


.. c:function:: char* ARKStepGetLinReturnFlagName(long int lsflag)

   Returns the name of the ARKLS constant corresponding to *lsflag*.

   **Arguments:**
      * *lsflag* -- a return flag from an ARKLS function.

   **Return value:**  The return value is a string containing the name of
   the corresponding constant. If using the ``SUNLINSOL_DENSE`` or
   ``SUNLINSOL_BAND`` modules, then if  1 :math:`\le` `lsflag`
   :math:`\le n` (LU factorization failed), this routine returns "NONE".

   .. warning::

      The user is responsible for freeing the returned string.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetLinReturnFlagName` instead.



.. c:function:: int ARKStepGetMassWorkSpace(void* arkode_mem, long int* lenrwMLS, long int* leniwMLS)

   Returns the real and integer workspace used by the ARKLS mass matrix linear solver interface.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *lenrwMLS* -- the number of ``sunrealtype`` values in the ARKLS mass solver workspace.
      * *leniwMLS* -- the number of integer values in the ARKLS mass solver workspace.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Notes:**
      The workspace requirements reported by this routine
      correspond only to memory allocated within this interface and to
      memory allocated by the ``SUNLinearSolver`` object attached
      to it.  The template mass matrix allocated by the user outside
      of ARKLS is not included in this report.

      In a parallel setting, the above values are global (i.e. summed over all
      processors).

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetMassWorkSpace` instead.


.. c:function:: int ARKStepGetNumMassSetups(void* arkode_mem, long int* nmsetups)

   Returns the number of calls made to the ARKLS mass matrix solver
   'setup' routine; these include all calls to the user-supplied
   mass-matrix constructor function.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmsetups* -- number of calls to the mass matrix solver setup routine.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMassSetups` instead.


.. c:function:: int ARKStepGetNumMassMultSetups(void* arkode_mem, long int* nmvsetups)

   Returns the number of calls made to the ARKLS mass matrix 'matvec setup'
   (matrix-based solvers) routine.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmvsetups* -- number of calls to the mass matrix matrix-times-vector setup routine.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMassMultSetups` instead.


.. c:function:: int ARKStepGetNumMassMult(void* arkode_mem, long int* nmmults)

   Returns the number of calls made to the ARKLS mass matrix 'matvec'
   routine (matrix-based solvers) or the user-supplied *mtimes*
   routine (matris-free solvers).

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmmults* -- number of calls to the mass matrix solver matrix-times-vector routine.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMassMult` instead.


.. c:function:: int ARKStepGetNumMassSolves(void* arkode_mem, long int* nmsolves)

   Returns the number of calls made to the ARKLS mass matrix solver 'solve' routine.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmsolves* -- number of calls to the mass matrix solver solve routine.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMassSolves` instead.


.. c:function:: int ARKStepGetNumMassPrecEvals(void* arkode_mem, long int* nmpevals)

   Returns the total number of mass matrix preconditioner evaluations,
   i.e. the number of calls made to *psetup*.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmpevals* -- the current number of calls to *psetup*.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMassPrecEvals` instead.


.. c:function:: int ARKStepGetNumMassPrecSolves(void* arkode_mem, long int* nmpsolves)

   Returns the number of calls made to the mass matrix preconditioner
   solve function, *psolve*.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmpsolves* -- the number of calls to *psolve*.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMassPrecSolves` instead.


.. c:function:: int ARKStepGetNumMassIters(void* arkode_mem, long int* nmiters)

   Returns the cumulative number of mass matrix solver iterations.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmiters* -- the current number of mass matrix solver linear iterations.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMassIters` instead.


.. c:function:: int ARKStepGetNumMassConvFails(void* arkode_mem, long int* nmcfails)

   Returns the cumulative number of mass matrix solver convergence failures.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmcfails* -- the current number of mass matrix solver convergence failures.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMassConvFails` instead.


.. c:function:: int ARKStepGetNumMTSetups(void* arkode_mem, long int* nmtsetup)

   Returns the cumulative number of calls made to the user-supplied
   mass-matrix-vector product setup function, *mtsetup*.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *nmtsetup* -- the current number of calls to *mtsetup*.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Note:** This is only accumulated for the "life" of the linear
   solver object; the counter is reset whenever a new mass-matrix
   linear solver module is "attached" to ARKStep, or when ARKStep is
   resized.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetNumMTSetups` instead.


.. c:function:: int ARKStepGetLastMassFlag(void* arkode_mem, long int* mlsflag)

   Returns the last return value from an ARKLS mass matrix interface routine.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *mlsflag* -- the value of the last return flag from an ARKLS
        mass matrix solver interface function.

   **Return value:**
      * *ARKLS_SUCCESS* if successful
      * *ARKLS_MEM_NULL* if the ARKStep memory was ``NULL``
      * *ARKLS_LMEM_NULL* if the linear solver memory was ``NULL``

   **Notes:**
      The values of *msflag* for each of the various solvers
      will match those described above for the function
      :c:func:`ARKStepGetLastLinFlag()`.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeGetLastMassFlag` instead.



.. _ARKODE.Usage.ARKStep.ARKStepExtraOutputs:

General usability functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. c:function:: int ARKStepWriteParameters(void* arkode_mem, FILE *fp)

   Outputs all ARKStep solver parameters to the provided file pointer.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *fp* -- pointer to use for printing the solver parameters.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      The *fp* argument can be ``stdout`` or ``stderr``, or it
      may point to a specific file created using ``fopen``.

      When run in parallel, only one process should set a non-NULL value
      for this pointer, since parameters for all processes would be
      identical.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeWriteParameters` instead.


.. c:function:: int ARKStepWriteButcher(void* arkode_mem, FILE *fp)

   Outputs the current Butcher table(s) to the provided file pointer.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *fp* -- pointer to use for printing the Butcher table(s).

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL* if the ARKStep memory was ``NULL``

   **Notes:**
      The *fp* argument can be ``stdout`` or ``stderr``, or it
      may point to a specific file created using ``fopen``.

      If ARKStep is currently configured to run in purely explicit or
      purely implicit mode, this will output a single Butcher table;  if
      configured to run an ImEx method then both tables will be output.

      When run in parallel, only one process should set a non-NULL value
      for this pointer, since tables for all processes would be
      identical.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKStepGetCurrentButcherTables` and :c:func:`ARKodeButcherTable_Write`
      instead.



.. _ARKODE.Usage.ARKStep.Reinitialization:

ARKStep re-initialization function
-------------------------------------

To reinitialize the ARKStep module for the solution of a new problem,
where a prior call to :c:func:`ARKStepCreate` has been made, the
user must call the function :c:func:`ARKStepReInit()`.  The new
problem must have the same size as the previous one.  This routine
retains the current settings for all ARKstep module options and
performs the same input checking and initializations that are done in
:c:func:`ARKStepCreate`, but it performs no memory allocation as it
assumes that the existing internal memory is sufficient for the new
problem.  A call to this re-initialization routine deletes the
solution history that was stored internally during the previous
integration, and deletes any previously-set *tstop* value specified via a
call to :c:func:`ARKStepSetStopTime()`.  Following a successful call to
:c:func:`ARKStepReInit()`, call :c:func:`ARKStepEvolve()` again for
the solution of the new problem.

The use of :c:func:`ARKStepReInit()` requires that the number of
Runge--Kutta stages, denoted by *s*, be no larger for the new problem than
for the previous problem.  This condition is automatically fulfilled
if the method order *q* and the problem type (explicit, implicit,
ImEx) are left unchanged.

When using the ARKStep time-stepping module, if there are changes to
the linear solver specifications, the user should make the appropriate
calls to either the linear solver objects themselves, or to the
ARKLS interface routines, as described in
:numref:`ARKODE.Usage.ARKStep.LinearSolvers`. Otherwise, all solver inputs set
previously remain in effect.

One important use of the :c:func:`ARKStepReInit()` function is in the
treating of jump discontinuities in the RHS functions.  Except in cases
of fairly small jumps, it is usually more efficient to stop at each
point of discontinuity and restart the integrator with a readjusted
ODE model, using a call to :c:func:`ARKStepReInit()`.  To stop when
the location of the discontinuity is known, simply make that location
a value of ``tout``.  To stop when the location of the discontinuity
is determined by the solution, use the rootfinding feature.  In either
case, it is critical that the RHS functions *not* incorporate the
discontinuity, but rather have a smooth extension over the
discontinuity, so that the step across it (and subsequent rootfinding,
if used) can be done efficiently.  Then use a switch within the RHS
functions (communicated through ``user_data``) that can be flipped
between the stopping of the integration and the restart, so that the
restarted problem uses the new values (which have jumped).  Similar
comments apply if there is to be a jump in the dependent variable
vector.


.. c:function:: int ARKStepReInit(void* arkode_mem, ARKRhsFn fe, ARKRhsFn fi, sunrealtype t0, N_Vector y0)

   Provides required problem specifications and re-initializes the
   ARKStep time-stepper module.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *fe* -- the name of the C function (of type :c:func:`ARKRhsFn()`)
        defining the explicit portion of the right-hand side function in
        :math:`M\, \dot{y} = f^E(t,y) + f^I(t,y)`.
      * *fi* -- the name of the C function (of type :c:func:`ARKRhsFn()`)
        defining the implicit portion of the right-hand side function in
        :math:`M\, \dot{y} = f^E(t,y) + f^I(t,y)`.
      * *t0* -- the initial value of :math:`t`.
      * *y0* -- the initial condition vector :math:`y(t_0)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_MEM_FAIL*  if a memory allocation failed
      * *ARK_ILL_INPUT* if an argument had an illegal value.

   **Notes:**
      All previously set options are retained but may be updated by calling
      the appropriate "Set" functions.

      If an error occurred, :c:func:`ARKStepReInit()` also
      sends an error message to the error handler function.





.. _ARKODE.Usage.ARKStep.Reset:

ARKStep reset function
----------------------

.. c:function:: int ARKStepReset(void* arkode_mem, sunrealtype tR, N_Vector yR)

   Resets the current ARKStep time-stepper module state to the provided
   independent variable value and dependent variable vector.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *tR* -- the value of the independent variable :math:`t`.
      * *yR* -- the value of the dependent variable vector :math:`y(t_R)`.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_MEM_FAIL*  if a memory allocation failed
      * *ARK_ILL_INPUT* if an argument had an illegal value.

   **Notes:**
      By default the next call to :c:func:`ARKStepEvolve()` will use the step size
      computed by ARKStep prior to calling :c:func:`ARKStepReset()`. To set a
      different step size or have ARKStep estimate a new step size use
      :c:func:`ARKStepSetInitStep()`.

      All previously set options are retained but may be updated by calling
      the appropriate "Set" functions.

      If an error occurred, :c:func:`ARKStepReset()` also sends an error message to
      the error handler function.

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeReset` instead.



.. _ARKODE.Usage.ARKStep.Resizing:

ARKStep system resize function
-------------------------------------

.. c:function:: int ARKStepResize(void* arkode_mem, N_Vector yR, sunrealtype hscale, sunrealtype tR, ARKVecResizeFn resize, void* resize_data)

   Re-sizes ARKStep with a different state vector but with comparable
   dynamical time scale.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *yR* -- the newly-sized state vector, holding the current
        dependent variable values :math:`y(t_R)`.
      * *hscale* -- the desired time step scaling factor (i.e. the next
        step will be of size *h\*hscale*).
      * *tR* -- the current value of the independent variable
        :math:`t_R` (this must be consistent with *yR*).
      * *resize* -- the user-supplied vector resize function (of type
        :c:func:`ARKVecResizeFn()`.
      * *resize_data* -- the user-supplied data structure to be passed
        to *resize* when modifying internal ARKStep vectors.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_NULL*  if the ARKStep memory was ``NULL``
      * *ARK_NO_MALLOC* if *arkode_mem* was not allocated.
      * *ARK_ILL_INPUT* if an argument had an illegal value.

   **Notes:**
      If an error occurred, :c:func:`ARKStepResize()` also sends an error
      message to the error handler function.

      If inequality constraint checking is enabled a call to
      :c:func:`ARKStepResize()` will disable constraint checking. A call
      to :c:func:`ARKStepSetConstraints()` is required to re-enable constraint
      checking.

   **Resizing the linear solver:**
      When using any of the SUNDIALS-provided linear solver modules, the
      linear solver memory structures must also be resized.  At present,
      none of these include a solver-specific "resize" function, so the linear
      solver memory must be destroyed and re-allocated **following** each
      call to :c:func:`ARKStepResize()`.  Moreover, the existing ARKLS
      interface should then be deleted and recreated by attaching the
      updated ``SUNLinearSolver`` (and possibly ``SUNMatrix``) object(s)
      through calls to
      :c:func:`ARKStepSetLinearSolver()`, and
      :c:func:`ARKStepSetMassLinearSolver()`.

      If any user-supplied routines are provided to aid the linear solver
      (e.g. Jacobian construction, Jacobian-vector product,
      mass-matrix-vector product, preconditioning), then the corresponding
      "set" routines must be called again **following** the solver
      re-specification.

   **Resizing the absolute tolerance array:**
      If using array-valued absolute tolerances, the absolute tolerance
      vector will be invalid after the call to :c:func:`ARKStepResize()`, so
      the new absolute tolerance vector should be re-set **following** each
      call to :c:func:`ARKStepResize()` through a new call to
      :c:func:`ARKStepSVtolerances()` and possibly
      :c:func:`ARKStepResVtolerance()` if applicable.

      If scalar-valued tolerances or a tolerance function was specified
      through either :c:func:`ARKStepSStolerances()` or
      :c:func:`ARKStepWFtolerances()`, then these will remain valid and no
      further action is necessary.

   **Example codes:**
      * ``examples/arkode/C_serial/ark_heat1D_adapt.c``

   .. deprecated:: 6.1.0

      Use :c:func:`ARKodeResize` instead.


.. _ARKStep_CInterface.MRIStepInterface:

Interfacing with MRIStep
------------------------

When using ARKStep as the inner (fast) integrator with MRIStep, the
utility function :c:func:`ARKStepCreateMRIStepInnerStepper` should be used to
wrap an ARKStep memory block as an :c:type:`MRIStepInnerStepper`.

.. c:function:: int ARKStepCreateMRIStepInnerStepper(void *inner_arkode_mem, MRIStepInnerStepper *stepper)

   Wraps an ARKStep memory block as an :c:type:`MRIStepInnerStepper` for use
   with MRIStep.

   **Arguments:**
      * *arkode_mem* -- pointer to the ARKStep memory block.
      * *stepper* -- the :c:type:`MRIStepInnerStepper` object.

   **Return value:**
      * *ARK_SUCCESS* if successful
      * *ARK_MEM_FAIL* if a memory allocation failed
      * *ARK_ILL_INPUT* if an argument had an illegal value.

   **Example usage:**
      .. code-block:: C

         /* fast (inner) and slow (outer) ARKODE objects */
         void *inner_arkode_mem = NULL;
         void *outer_arkode_mem = NULL;

         /* MRIStepInnerStepper to wrap the inner (fast) ARKStep object */
         MRIStepInnerStepper stepper = NULL;

         /* create an ARKStep object, setting fast (inner) right-hand side
            functions and the initial condition */
         inner_arkode_mem = ARKStepCreate(ffe, ffi, t0, y0, sunctx);

         /* setup ARKStep */
         . . .

         /* create MRIStepInnerStepper wrapper for the ARKStep memory block */
         flag = ARKStepCreateMRIStepInnerStepper(inner_arkode_mem, &stepper);

         /* create an MRIStep object, setting the slow (outer) right-hand side
            functions and the initial condition */
         outer_arkode_mem = MRIStepCreate(fse, fsi, t0, y0, stepper, sunctx);

   **Example codes:**
      * ``examples/arkode/CXX_parallel/ark_diffusion_reaction_p.cpp``

   .. deprecated:: 6.2.0

      Use :c:func:`ARKodeCreateMRIStepInnerStepper` instead.
