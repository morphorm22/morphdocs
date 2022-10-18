.. _examples_intro_sec:

Introduction
############

This chapter will introduce the reader to the optimization-based design capabilities in
Morphorm through a series of tutorials. Each tutorial will cover the steps to sucessfully
perform design optimization studies. The tutorials will introduce topology and shape
optimization concepts to the readers while demonstrating how to apply these concepts to
perform design optimization studies.

.. Tutorials on Design of Experiment, Surrogate-Based Optimization, and Pareto Optimization
.. are also presented in this chapter. Furthermore, this chapter introduces readers to Design
.. Under Uncertainty concepts and shows readers how design under uncertainty methods can be
.. applied to produce design solutions robust to inherent imperfections. Another key feature
.. of Morphorm is its ability to combine multiple optimization-based design methods to build
.. automated end-to-end design workflows. This chapter will present readers several tutorials
.. explaining how one can combine multiple Morphorm optimization-based design pardigms to
.. produce hands-off end-to-end design workflows.

.. _examples_topt_sec:

Topology Optimization
#####################

Topology optimization is a mathematical method used to optimize the material layout of 
a physical system, or component, given a set of mathematical objective and constraint
functions. Topology optimization is not shape optimization, which will be introduced in 
a subsequent subsection, since in topology optimization the algorithm is free to choose any
material layout within the design space that satisfies the design objectives and constraints. 
In contrast, shape optimization methods operate on a restricted set of geometric features,
i.e. parameters, that define a preset configuration.

A topology optimization problem can be written in the general form of an optimization 
problem as:

.. math::
   :label: eq_topo_form

   \min_{z} \quad & \sum_{i=1}^{N_f} f_i(u(z),z) \\
   \textrm{s.t.} \quad & g_j(u(z),z) \leq 0\quad\mbox{with}\ j=1,\dots,N_g \\
   \quad & z_{min}\leq z \leq z_{max}

The general topology optimization problem statement introduced in :math:numref:`eq_topo_form` includes: 

* :math:`N_f` objective functions :math:`f_i(u(z),z)\ \mbox{with}\ i=1,\dots,N_f`. The
  objective functions represent the quantities that are being minimized, or maximized,
  to reach the best design performance.
* The material distribution is denoted with design variable :math:`z`. The design variable
  is defined by the density of the material at each location in the case of a density based
  topology optimization formulation or by a level set field in the case of level set based
  topology optimization formulation.
* :math:`N_g` constraint functions :math:`g_j(u(z),z)`. The constraint functions represent the
  design requirements that the optimized design must satisfy.
* The quantities :math:`z_{max}` and :math:`z_{min}` in :math:numref:`eq_topo_form` respectively 
  denote upper and lower bounds on the design variable :math:`z`.

Evaluating :math:`u(z)` requires solving a differential equation using a method for
numerically solving differential equations, e.g. `finite element method <https://en.wikipedia.org/wiki/Finite_element_method>`_
and `finite volume method <https://en.wikipedia.org/wiki/Finite_volume_method>`_, since these 
equations do not have a known analytical solution.
  
.. _examples_topt_material_discretization_subsec:

Material Description
********************

Ideally, in a topology optimization problem, the design variables should be modeled as a discrete 
variable that only can take on a zero or one value at a material point. A value of one indicates
the existance of material and zero the abscence of material. However, discrete optimization methods
are ill-suited for relevant engineering applicaitons due to the computational demand associated
with discrete optimization methods. To circumvent this hurdle, a continuum design variable
description is used to model the material distribution within the design domain. This approach,
combined with the adjoint method, permits the application of gradient-based optimization methods
to solve :math:numref:`eq_topo_form`. Gradient-based optimization methods are more effective
than discrete optimization methods in finding a solution to design optimization problems with a
large number of design variables such as :math:numref:`eq_topo_form`.

The Morphorm topology optimization solvers are capable of handling both density and level
set material descriptions. A combination of material interpolation function, filters, and 
projection methods are often used in density-based topology optimization problems to steer
the algorithm towards a crisp "0-1" design solution. In contrast, with level set methods,
an accurate design description can be realized at each optimization iteration. Therefore,
additional verification simulations to verify performance metrics and physical responses
are not needed with level set methods once an optimized solution is reached. However, in
some instances, e.g., compliance minimization, a density-based approach may reach a solution
faster than a level set approach.

The tutorials presented in this section describe how to perform topology optimization studies
with density and level set methods. The tutorials presented in this capter cover a spectrum of
applications, from standard single-physics applications to complex multi-physics applications.
Each topology optimization approach requires additional methods to steer the optimizer towards
a configuration that achieves the best performance given the design requirements for
:math:numref:`eq_topo_form`. Readers will be able to setup and perform density- and level set based
topology optimization problems after going through the tutorials.

.. _examples_topt_structTO_density_subsec:

Density Method
**************

A critical aspect of density-based topology optimization methods is the selection of the material 
interpolation function, which is used to steer the optimizer towards a "0-1" design solution.
In a structural topology optimization problem applying density-based topology optimization methods,
the density values are set to :math:`0\leq{z}_{min}\leq{z}\leq{1}`, where :math:`0` denotes the
absence of material at a given material point and :math:`1` denotes the existence of material
at a material point. Morphorm applies a modified Solid Isotropic Material Penalization material
interpolation method, which is defined as

.. math::
   :label: eq_modified_simp

   z_{min} + (1 + z_{min})z^p

where :math:`z_{min}` is the :ref:`minimum value the density <input_deck_options_scenario_minersatz_kw>`  
can take at a given material point. The :math:`z_{min}` parameter is used to prevent singular matrices 
and thus singular linear system of equations. The parameter :math:`p` denotes a :ref:`penalization 
factor <input_deck_options_scenario_pexp_kw>`, which usually takes on the value of 3. In some applications,
such as stress constrained mass minimization problems, a continuation scheme is used on the penalization
factor to steer the optimizer towards a "0-1" design solution. Morphorm applies a :ref:`filter <input_deck_options_method_filter_kws>` to avoid numerical artifacts that result from the
discretization of :math:`z` with an unstable finite element formulation. The filter also provides a
mechanism to implicitly enforce an approximate minimum feature size constraint on the topology
optimization solution. While the filter does not completely eliminates the issue of mesh-dependencies,
it greatly helps control it. A mathematical description of the filter is provided in the next sections.

.. _examples_topt_structTO_density_filter_subsubsec:

Kernel Filter
=============

There are two types of filters implemented in Morphorm. The first is the kernel filter, which can 
take on multiple variations. A linear kernel filter is mathematically defined as

.. math::
   :label: eq_linear_kernel_filter

   F_{ij}=\max\left(0,1-\frac{d(i,j)}{R}\right)
    
where :math:`R` is the :ref:`filter radius <input_deck_options_method_filter_radius_kw>`, 
:math:`d(i,j)` is the distance between material points :math:`z^m_i` and :math:`z^m_j` for 
candidate material :math:`m`. The filtered material points :math:`\hat{z}^m_i` for candidate
material :math:`m` are given as

.. math::
   :label: eq_filtered_material_field

   \hat{z}^m_j=\sum_{i=1}^{N_p}=w_{ij}z_i^m
   
where :math:`N_p` denotes the number of material points inside the filter radius and the weights 
:math:`w_{ij}` are defined as

.. math::
   :label: eq_kernel_filter_weights

   w_{ij}=\frac{F_{ij}}{\sum_{k\in\mathcal{N}_j}F_{kj}}

:math:`\mathcal{N}_j=\{x_i^m\colon{d}(i,j)\leq{R}\}` is the neighborhood of material points 
inside the filter radius :math:`R`, which includes the material points on the boundary of the 
radius, with respect to material point :math:`x_j^m`. The other filter supported in Morphorm
is the Helmholtz filter, which is covered in this :ref:`section  <examples_topt_structTO_density_helmholtz_subsubsec>`.
Readers are advice to review the :ref:`filter section <input_deck_options_method_filter_kws>`
to learn how to best set the kernel filter parameters.

.. _examples_topt_structTO_density_helmholtz_subsubsec:

Helmholtz Filter
================

The Helmholtz filter, also know as the PDE filter, is another efficient way of enforcing an 
approximate minimum feature size constraint in topology optimization problems. Traditionally,
the Helmholtz filter has been formulated as the minimization of the potential :math:`\Pi`:

.. math::
   :label: eq_helmholtz_filter_potential
   
   \Pi(\hat{z})=\frac{1}{2}\int_{\Omega}\ell_0^2\Vert\nabla\hat{z}\Vert^2\ d\Omega + \frac{1}{2}
   \int_{\Omega}\left(z-\hat{z}\right)d\Omega

where :math:`\ell_0` is a length scale parameter. The second integral above aims to keep the 
filtered filtered design variables :math:`\hat{z}` close to the unfiltered design variables
:math:`z`, i.e. the filtered design variables should not be significantly different than the
unfiltered design variables.However, the unfiltered design variables can be highly oscillatory, 
which the first integral in :math:numref:`eq_helmholtz_filter_potential` aims to control. The 
compromise between these two goals is regulated by the lenght scale parameter :math:`\ell_0`.

Minimizing the potential in :math:numref:`eq_helmholtz_filter_potential` with respect to the 
filtered design variables :math:`\hat{z}` yields 

.. math::
   :label: eq_helmholtz_filter_1
   
   \delta\Pi(\hat{z})&=-\int_{\Omega}\ell_0^2\Delta\hat{z}\delta\hat{z}d\Omega+\int_{\Omega}\hat{z}\delta
   \hat{z}-\int_{\Omega}z\delta\hat{z}\ = 0 \quad\mbox{in}\ \Omega \\ \\
   \mbox{with:}&\quad\nabla\hat{z}\cdot\mathbf{n}=0\quad \mbox{on}\ \Gamma

where :math:`\mathbf{n}` is the outward normal unit vector to the design volume :math:`\Omega`.
The Helmholtz filter formulation in :math:numref:`eq_helmholtz_filter_potential` has one drawback,
it does not penalize placement of material along the boundaries of the optimized design. This causes
the Helmholtz filter to favor optimized designs with boundaries coinciding with the the boundaries
of the allowable design domain. This undesired behavior is know as the "stick" effect. Luckily,
this problem can be mitigated by assigning a cost to the material placed along the boundaries
of the allowable design domain. To achieve this goal, a boundary integral is added to
:math:numref:`eq_helmholtz_filter_potential`, which yields

.. math::
   :label: eq_helmholtz_filter_potential_wboundary
   
   \tilde{\Pi}(\hat{z})=\frac{1}{2}\int_{\Omega}\ell_0^2\Vert\nabla\hat{z}\Vert^2\ d\Omega + \frac{1}{2}
   \int_{\Omega}\left(z-\hat{z}\right)d\Omega + \frac{1}{2}\int_{\Gamma}\ell_s\hat{z}^2 d\Gamma

where :math:`\ell_s` is the surface length scale parameter and :math:`\Gamma` denotes the domain 
boundary. Minimizing :math:numref:`eq_helmholtz_filter_potential_wboundary` with respect to the 
filtered design variables :math:`\hat{z}` yields 

.. math::
   :label: eq_helmholtz_filter_2
   
   \delta\tilde{\Pi}(\hat{z})&=-\int_{\Omega}\ell_0^2\Delta\hat{z}\delta\hat{z}d\Omega-\int_{\Omega}(z-\hat{z})
   \delta\hat{z}d\Omega + \int_{\Gamma}(\ell_0^2\nabla\hat{z}\cdot\mathbf{n}+\ell_s\hat{z})d\Gamma=0 \\ \\
   \mbox{with:}&\quad\ell_0^2\nabla\hat{z}\cdot\mathbf{n}=-\ell_s\hat{z}\quad \mbox{on}\ \Gamma
   
Instead of solving :math:numref:`eq_helmholtz_filter_1` with homogeneous boundary conditions 
to enforce an approximate minimum feature size constraint, :math:numref:`eq_helmholtz_filter_2`
is solved with Robin boundary conditions :math:`\ell_0^2\nabla\hat{z}\cdot\mathbf{n}=-\ell_s
\hat{z}\ \mbox{on}\ \Gamma`. If :math:`\ell_s\rightarrow{0}`, :math:numref:`eq_helmholtz_filter_1`
is recovered. In contrast, placing material along the domain boundaries becomes constly as
:math:`\ell_s\rightarrow\infty` and designs that adhere to the boundaries of the allowable
design domain are avoided. Readers are advice to review this :ref:`section
<input_deck_options_method_filter_kws>` to learn how to best set the Helmholtz filter
parameters.    


.. _examples_topt_structTO_density_projection_subsubsec:

Heaviside Projection 
====================

In addition to material interpolation functions and density filters, density-based topology
optimization problems may require the use of projection techniques to steer the optimizer
towards a "0-1" design solution. The filter can create transition regions with intermediate
pseudo-density values. In order to mitigate and avoid the transition regions, a projection
scheme is employed. Morphorm applies a heaviside projection function of the form

.. math::
   :label: eq_proj_func

   \bar{z}^m_j=\frac{\tanh(\beta\eta) + \tanh(\beta(\hat{z}^m_j-\eta))}{\tanh(\beta\eta) + \tanh(\beta(1-\eta))}

where :math:`\eta` governs the density threshold at which the projection takes place and 
:math:`\beta` governs the strength of the projection operation. The :math:`\bar{z}^m_j` are 
the projected material points for candidate material :math:`m`. The parameter :math:`\eta`
is set to its default value of 0.5 while a continuation scheme is used to update :math:`\beta`.
:math:`\beta` can be incrementally increased at a fixed frequency to steer the optimizer to
a "0-1" design solution. Readers are advice to review the :ref:`filter section
<input_deck_options_method_filter_kws>` to learn how to best set the parameters for the
heaviside projection scheme presented in :math:numref:`eq_proj_func`.

.. _examples_topt_structTO_density_fixedblocks_subsubsec:

Fixed Blocks 
************

In topology optimization problems, the optimization algorithm is capable of adding or removing 
material at every material point within the design domain :math:`\Omega`. In some use cases, the 
designer may want to discourage the optimization algorithm from removing material from certain 
regions due to practical engineering considerations. For instance, a component must be mounted on 
top of a surface. Therefore, the optimization algorithm cannot be allowed to remove material available
on this surface as well as other close surrounding areas. The fixed block feature is available in 
Morphorm to enable users to specify non-optimizable regions in the design domain. This information 
is pass to the optimization algorithm and at runtime the algorithm avoids removing material from
the non-optimizable regions. The :ref:`fixed_block_ids <input_deck_options_method_fblocks_ids_kw>` 
parameter in the input deck enables users to specify the element block(s) associated with these 
non-optimizable regions. The fixed block feature will be utilized in the upcoming tutorials to 
show how to properly set and use the fixed block functionality.

.. _examples_topt_compliance_sec:

Compliance Minimization
#######################

The most common topology optimization problem solved by practitioners is complaince minimization. 
A compliance minimization problem seeks to minimize the structural compliance, i.e., maximize the
stiffness of the structure, given a volume or mass constraint. In this tutorial, a density-based
topology optimization approach is applied. At the end of this tutorial, users will be able to set
and solve density-based compliance minization problems.

Mathematically, a compliance minimization problem is defined as:

.. math::
   :label: eq_compliance_prob

   \min_{\mathbf{z}} \quad & \frac{1}{2}\mathbf{f}^T \mathbf{u}(\hat{\mathbf{z}}) \\
   \textrm{s.t.} \quad & V(\hat{\mathbf{z}}) \leq V_{t} \\
   \quad & \mathbf{z}_{min}\leq \mathbf{z} \leq \mathbf{z}_{max} \\ \\
   \textrm{with:} \quad & \mathbf{u}(\hat{\mathbf{z}})=\mathbf{K}^{-1}(\hat{\mathbf{z}})\mathbf{f}

Evaluating the displacements :math:`\mathbf{u}(\hat{\mathbf{z}})` requires solving the classic
linear elastostatics problem :math:`\mathbf{K}(\hat{\mathbf{z}})\mathbf{u} - \mathbf{f} = 0`,
where :math:`\mathbf{K}(\hat{\mathbf{z}})` is the stiffness matrix, which depends on the vector
of :ref:`filtered design variables <examples_topt_structTO_density_filter_subsubsec>`
:math:`\hat{\mathbf{z}}`, and :math:`\mathbf{f}` is the force vector. :math:`V_t` is the target
volume (or mass) while :math:`V(\hat{\mathbf{z}})` denotes the current design volume (or mass).
The superscript :math:`T` in :math:numref:`eq_compliance_prob` denotes the transpose operation. 

.. _examples_topt_compliance_inputdeck_subsec:

Input Deck 
**********

The following excerpt shows the input deck used to solve the compliance minimization problem
defined in :math:numref:`eq_compliance_prob`.

.. code-block:: console
   
  begin service 1
    code platomain
    number_processors 1
  end service

  begin service 2
    code plato_analyze
    number_processors 1
    device_ids 0
  end service
   
  begin criterion 1
    type mechanical_compliance 
  end criterion
 
  begin criterion 2
    type volume 
  end criterion
      
  begin scenario 1
    physics steady_state_mechanics
    dimensions 3
    loads 1
    boundary_conditions 1
    material 1
  end scenario   

  begin objective
    type weighted_sum
    criteria 1 
    services 2 
    scenarios 1 
    weights 1
  end objective

  begin output
    service 2
    data dispx dispy dispz vonmises
  end output

  begin boundary_condition 1
    type fixed_value
    location_type nodeset
    location_name ns1
    degree_of_freedom dispx dispy dispz
    value 0 0 0
  end boundary_condition

  begin load 1
    type traction
    location_type sideset
    value 0 0 -1e5
    location_name ss1
  end load
         
  begin constraint 1
    criterion 2
    relative_target .15
    service 1
    scenario 1
  end constraint
   
  begin material 1
    material_model isotropic_linear_elastic
    poissons_ratio .33
    youngs_modulus 1e9
  end material

  begin block 1
    material 1
  end block
  
  begin block 2
    material 1
  end block 2

  begin optimization_parameters
    max_iterations 50
    filter_type helmholtz
    filter_radius_absolute 0.173
    boundary_sticking_penalty 0.0
    fixed_block_ids 2
    optimization_algorithm oc
    enforce_bounds false
    output_frequency 0
  end optimization_parameters
   
  begin mesh
    name FixedBlocks.exo
  end mesh

.. _examples_topt_compliance_results_subsec:

Results 
*******

.. _examples_topt_stressconst_sec:

Stress Constrained Mass Minimization
####################################

The consideration of stress constraints in topology optimization formulations is
fundamental to the overall performance of the structure. Stiffness-based designs 
such as compliance minimization problems do not treat material strength as a driving 
design requirement in the topology optimization formulation. Therefore, stiffness-based 
formulations can yield innefective designs due to their inability to meet stress
requirements. From a structural engineering standpoint, a more appropriate topology 
optimization formulation should aim to find the lightest structure while not exceeding 
the material strength requirements at any material point.

Since stress is a fundamental local quantity, an effective topology optimization
formulation must enforce the strength requirement at every material point to prevent
yielding from happening. Such topology optimization formulation leads to a large
number of stress constraints, which is difficult to model with standard topology
optimization algorithms. Morphorm applies an augmented Lagrangian (AL) formulation
to effectively solve the topology optimization problem with local stress constraints.
The AL algorithm yields solutions that meet the stress requirement at every material
point in the design domain while modeling the local nature of stress. The effectiveness
of the algorithm results from its ability to model large number of constraints.

A stress-constrained topology optimization problem aims to find the lightest structure 
capable of supporting the applied loads without experiencing failure. To limit the
stress at points :math:`x_j\in\Omega`, stress constraints of the form :math:`g_j(u(z),z)
\leq{0},\ j=1,\dots,N_p` are imposed, where :math:`\Omega` is the design domain and
:math:`N_p` is the number of material points where stress constraints are applied. Thus,
in its discrete form, the stress constrained mass minimization topology optimization
problem is defined as:

.. math::
   :label: eq_stressconst_prob

   \min_{\mathbf{z}} \quad & \frac{\mathbf{V}^Tm_V(\hat{\mathbf{z}})}{\mathbf{V}^T\mathbf{1}} \\
   \textrm{s.t.} \quad & m_E(\hat{\mathbf{z}})\Lambda_j(u(\hat{\mathbf{z}}))\left(\Lambda_j^2(\mathbf{u}(\hat{\mathbf{z}}))+1\right) \leq 0,\quad j=1,\dots,N_p \\
   \quad & \mathbf{z}_{min}\leq \mathbf{z} \leq \mathbf{z}_{max} \\ \\
   \textrm{with:} \quad & \Lambda_j(\mathbf{u}(\hat{\mathbf{z}})=\frac{\sigma^v_j(\mathbf{u}(\hat{\mathbf{z}}))}{\sigma_{lim}}-1 \\
   \quad & \mathbf{u}(\hat{\mathbf{z}})=\mathbf{K}^{-1}(\hat{\mathbf{z}})\mathbf{f}

where :math:`\hat{\mathbf{z}}` are the :ref:`filtered designed variables <examples_topt_structTO_density_filter_subsubsec>`,
:math:`\sigma^v_j(\mathbf{u}(\hat{\mathbf{z}}))` is the Von Mises stress at the j-th
material point, and :math:`\sigma_{lim}` is the Von mises stress limit at all material
points in the domain. The volume interpolation function :math:`m_V(\hat{\mathbf{z}})` is
given by :math:numref:`eq_proj_func` while the Von mises stress interpolation function
:math:`m_E(\hat{\mathbf{z}})` is given by

.. math::
   :label: eq_vonmises_intrp
   
   \mathbf{z}_{min} + (1-\mathbf{z}_{min})[m_V(\hat{\mathbf{z}})]^p
   
where :math:`m_V(\hat{\mathbf{z}})` is given by :math:numref:`eq_proj_func` and :math:`p` is a
:ref:`penalization factor <examples_topt_structTO_density_subsec>`.

.. _examples_topt_stresscont_inputdeck_subsec:

Input Deck 
**********

The following excerpt shows the input deck used to solve the compliance minimization problem
defined in :math:numref:`eq_stressconst_prob`. 

.. code-block:: console

  begin service 1 
    code platomain 
    number_processors 1
  end service

  begin service 2
    code plato_analyze
    number_processors 1
    update_problem true
  end service
   
  begin criterion 1
    type stress_and_mass
    scmm_constraint_exponent 2
    stress_limit 3e6
    scmm_penalty_expansion_multiplier 1.5
    scmm_initial_penalty 1
    scmm_penalty_upper_bound 10000
  end criterion
 
  begin scenario 1
    physics steady_state_mechanics
    dimensions 3
    loads 1
    boundary_conditions 1
    material 1
  end scenario   

  begin objective
    type weighted_sum
    criteria 1 
    services 2 
    scenarios 1 
    weights 1
  end objective

  begin output
    service 2
    data dispx dispy dispz vonmises
  end output

  begin boundary_condition 1
    type fixed_value
    location_type nodeset
    location_name ss_1
    degree_of_freedom dispx dispy dispz
    value 0 0 0
  end boundary_condition

  begin load 1
    type traction
    location_type sideset
    location_name ss_2
    value 0 -1e6 0
  end load
            
  begin material 1
    material_model isotropic_linear_elastic
    poissons_ratio .3
    youngs_modulus 1e9
    mass_density 1e3
  end material

  begin block 1
    material 1
  end block

  begin optimization_parameters
    max_iterations 1
    filter_radius_absolute 0.05712
    number_buffer_layers 0
    verbose true
    write_restart_file false
    restart_iteration 0
    optimization_algorithm rol_bound_constrained
    rol_subproblem_model lin_more
    reset_algorithm_on_update true
    hessian_type zero
    problem_update_frequency 1
    output_frequency 500
  end optimization_parameters

  begin mesh
    name lbracket.exo
  end mesh

.. _examples_topt_stressconst_results_subsec:

Results 
*******

.. _examples_topt_fluids_pressdrop_sec:

Pressure Drop Minimization
##########################

The governing incompressible Navier-Stokes equations are defined as

:math:`\textit{Incompressibility Condition}`

.. math::
  :label: eq_incompressible_condition
  
  \frac{\partial u_i}{\partial x_i}=0

:math:`\textit{Mass Conservation}`

.. math::
  :label: eq_mass_conservation
  
  \frac{1}{c^2}\frac{\partial p}{\partial t}=-\rho_f\frac{\partial u_i}{\partial x_i}
  \quad\mbox{in}\quad\Omega

:math:`\textit{Momentum Conservation}`

.. math::
  :label: eq_momentum_conservation

  \rho_f\left[ \frac{\partial u_i}{\partial t} + \frac{\partial}{\partial x_j}(u_j u_i) \right] = 
  -\frac{\partial p}{\partial x_i} + \frac{\partial\tau_{ij}}{\partial x_j} + \frac{\mu}{\kappa} 
  u_i\quad\mbox{in}\quad\Omega

The domain :math:`\Omega` is defined as the union of the fluid and solid domains, :math:`\Omega=
\Omega_f\cup\Omega_s`. The term :math:`u_i` is the i-th velocity component, :math:`x_i` i-th spatial 
coordinate, :math:`\rho_f` is the fluid's density, :math:`p` is the pressure, :math:`c` is the speed 
of sound, :math:`t` denotes time, :math:`\mu` is the dynamic viscocity, :math:`\kappa` is the permeability 
coefficient. The :math:`\frac{\mu}{\kappa}` term is only used in density-based topology optimization 
problems. This term is not part of the momentum conservation equation :math:numref:`eq_momentum_conservation` 
in level-set based topology optimization problems.

The deviatoric stress tensor is defined as

.. math::
  :label: eq_deviatoric_stress_incompressible
  
  \tau_{ij}=\mu\left(\frac{\partial u_i}{\partial x_j}+\frac{\partial u_j}{\partial x_i}\right)

The conservation equations :math:numref:`eq_incompressible_condition` - :math:numref:`eq_momentum_conservation` 
are completed after defining the boundary conditions

.. math::
  :label: eq_initial_bcs
  
  u_i=u_i^0\quad\mbox{on}\quad\Gamma_u

and

.. math::
  :label: eq_traction_bcs
  
  t_i=\left( \tau_{ij} - \delta_{ij}p \right)n_j=t_i^0\quad\mbox{on}\quad\Gamma_t

where :math:`\Gamma_u` and :math:`\Gamma_t` denote the surfaces where the velocity and traction 
boundary conditions and :math:`n_j` is the outward normal. 

A pressure drop minimization problem aims to find the lightest structure capable of minimizing 
the pressure drop between the inlets and outlets of a system.  

.. math::
   :label: eq_pressure_drop

   \min_{z} \quad & \int_\Gamma\left( p_{in} - p_{out} \right) \\
   \textrm{s.t.} \quad & m_f(\hat{z}) \leq m_{t} \\
   \quad & z_{min}\leq z \leq z_{max}

where :math:`p` is computed by solving the conservation equations :math:numref:`eq_incompressible_condition` 
- :math:numref:`eq_momentum_conservation`. The :math:`m_f(\hat{z})` term is the mass of the fluid material 
and :math:`m_t` is the target mass. 

.. _examples_topt_fluids_pressdrop_inputdeck_subsec:

Input Deck 
**********

The following excerpt shows the input deck used to solve the pressure drop minimization problem

.. code-block:: console

  begin service 1
    code platomain
    number_processors 1
  end service

  begin service 2
    code plato_analyze
    number_processors 1
  end service

  begin output
    service 2
    native_service_output false
  end output

  begin criterion 1
    type composite
    criterion_ids 2 3
    criterion_weights 0.01 -0.01
  end criterion

  begin criterion 2
    type mean_surface_pressure
    location_name inlet
  end criterion

  begin criterion 3
    type mean_surface_pressure
    location_name outlet
  end criterion

  begin criterion 4
    type volume
  end criterion

  begin scenario 1
    physics steady_state_incompressible_fluids
    dimensions 2
    boundary_conditions 1 2 3 4 5
    material 1
    linear_solver_tolerance 1e-20
    linear_solver_iterations 1000
  end scenario

  begin objective
    scenarios 1
    criteria 1
    services 2
    type weighted_sum
    weights 1
  end objective

  begin constraint 1
    criterion 4
    relative_target 0.25
    type less_than
    service 1
    scenario 1
  end constraint

  begin boundary_condition 1
    type zero_value
    location_type nodeset
    location_name no_slip
    degree_of_freedom velx
  end boundary_condition

  begin boundary_condition 2
    type zero_value
    location_type nodeset
    location_name no_slip
    degree_of_freedom vely
  end boundary_condition

  begin boundary_condition 3
    type fixed_value
    location_type nodeset
    location_name inlet
    degree_of_freedom velx
    value 1.5
  end boundary_condition

  begin boundary_condition 4
    type fixed_value
    location_type nodeset
    location_name inlet
    degree_of_freedom vely
    value 0
  end boundary_condition

  begin boundary_condition 5
    type zero_value
    location_type nodeset
    location_name outlet
    degree_of_freedom press
  end boundary_condition

  begin block 1
    material 1
    name block_1
  end block

  begin material 1
    material_model laminar_flow
    reynolds_number 100
  end material

  begin optimization_parameters
    optimization_algorithm mma
    discretization density
    max_iterations 5
    mma_move_limit 0.25
    filter_radius_scale 1.75
  end optimization_parameters

  begin mesh
    name pipe_flow.exo
  end mesh

.. _examples_topt_fluids_pressdrop_results_subsec:

Results 
*******

.. _examples_topt_thermal_compliance_sec:

Thermal Compliance Minimization
###############################

Thermal analyses are used to determine the temperature field and heat fluxes of a structure. Thermal 
analyses are widely used in engineering practices such as aerospace vehicle design, electronics cooling 
system design and automotive design. The governing equation for linear steady state thermal analysis 
is given as

.. math::
  :label: eq_thermal_steady_state
  
Mathematically, a thermal compliance minimization problem is defined as:
