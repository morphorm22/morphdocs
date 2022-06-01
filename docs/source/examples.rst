.. _examples_intro_sec:

Introduction
############

This chapter will introduce the reader to the optimization-based design capabilities in
Morphorm through a series of tutorials. Each tutorial will cover all the steps required
to sucessfully run a Morphorm optimization-based design problem. The tutorials will
introduce readers to topology and shape optimization concepts and show them how to apply 
these concepts to solve single- and multi-physics optimization-based design problems.
Tutorials on Design of Experiment, Surrogate-Based Optimization, and Pareto Optimization
are also presented in this chapter. Furthermore, this chapter introduces readers to Design
Under Uncertainty concepts and shows readers how design under uncertainty methods can be 
applied to produce design solutions robust to inherent imperfections. Another key feature 
of Morphorm is its ability to combine multiple optimization-based design methods to build 
automated end-to-end design workflows. This chapter will present readers several tutorials 
explaining how one can combine multiple Morphorm optimization-based design pardigms to 
produce hands-off end-to-end design workflows.

.. _examples_topt_sec:

Topology Optimization
#####################

Topology optimization is a mathematical method used to optimize the material layout of 
a physical system, or component, given a set of mathematical objectives and constraint 
functions. Topology optimization is not shape optimization, which will be introduced in 
a subsequent subsection, since the topology optimization algorithm is free to choose any 
material layout within the design space that satisfies the design objectives and constraints. 
In contrast, the shape optimization algorithm is only allowed to operate on a restricted set 
of geometric features, i.e. parameters, that define a predefined configuration. 

A topology optimization problem can be written in the general form of an optimization 
problem as:

.. math::
   :label: eq_topo_form

   \min_{z} \quad & \sum_{i=1}^{N_f} f_i(u(z),z) \\
   \textrm{s.t.} \quad & g_j(u(z),z) \leq 0\quad\mbox{with}\ j=1,\dots,N_g \\
   \quad & z_{min}\leq z \leq z_{max}

The general topology optimization problem statement introduced in :math:numref:`eq_topo_form` includes: 

* :math:`N_f` objective functions :math:`f_i(u(z),z)`. The objective functions represent the 
  quantities that are being minimized, or maximized, to achieve the best performance of the 
  optimized design solution.
* The material distribution as a design variable :math:`z`. This design variable is defined by 
  the density of the material at each location in the case of density-based topology optimization 
  problems or by a level set field in the case of level set based topology optimization problems. 
* :math:`N_g` constraint functions :math:`g_j(u(z),z)`. The constraint functions represent the
  design requirements that the optimized design solution must satisfy.  
* The quantities :math:`z_{max}` and :math:`z_{min}` in :math:numref:`eq_topo_form` respectively 
  denote upper and lower bounds on the design variable :math:`z`.

Evaluating :math:`u(z)` requires solving a differential equation using a method for numerically 
solving differential equations, e.g. `finite element method <https://en.wikipedia.org/wiki/Finite_element_method>`_ 
and `finite volume method <https://en.wikipedia.org/wiki/Finite_volume_method>`_, since these 
equations do not have a known analytical solution.
  
.. _examples_topt_material_discretization_subsec:

Material Description
********************

Ideally, the design variable :math:`z` should be modeled as a discrete design variable that only 
takes on a zero or one value at a given material point, where one indicates the existance of 
material and zero the abscence of material. However, this approach demands the application of discrete 
optimization methods to solve :math:numref:`eq_topo_form`, which are ill-suited for most relevant 
engineering applications. To circumvent this hurdle, a continuum design variable description is 
used to model the material distribution within the design domain. This approach permits the 
application of gradient-based optimization methods to solve :math:numref:`eq_topo_form`, which are 
computationally more effective than discrete optimization methods in finding a solution to 
:math:numref:`eq_topo_form`. 

The topology optimization solvers in Morphorm are capable of handling both density and level 
set material descriptions. A combination of material interpolation function, filters, and 
projection methods are often used in density-based topology optimization problems to steer 
the algorithm towards a crisp "0-1" design solution. In contrast, with level set methods, an 
explicit design configuration can be achieved at each optimization iteration. Thus, eliminating 
the requirement of performing additional numerical studies after finding an optimized design
solution in order to verify the optimized performance metrics and physical quantities of interests. 
However, in certain use cases (compliance minimization), density-based methods 
may reach an optimized design solution faster than level set methods. 

The tutorials presented in this section describe how to set a topology optimization problem
using density or level set material descriptions. These tutorials will cover a spectrum of 
applications, from the most common single-physics applications to the most complex multi-physics 
applications. Each material description requires additional methods to help steer the optimization 
algorithm towards a design configuration that achieves the best performance given the design 
requirements for :math:numref:`eq_topo_form`. These tutorials will describe how to set and tune 
the parameters associated with these methods based on the choosen material description. Readers 
should be capable of setting both density- or level set based topology optimization problems 
for a plethora of engineering applications after going through the tutorials presented herein. 

.. _examples_topt_structTO_density_subsec:

Density Method
**************

A critical aspect of density-based topology optimization methods is the selection of the material 
interpolation function, which is used to aid steer the optimizer towards a "0-1" design solution. 
In a density-based topology optimization problem, the density values are set to :math:`0\leq{z}_{min}
\leq{z}\leq{1}`, where :math:`0` denotes the absence of material at a given material point and 
:math:`1` denotes the existence of material at a given material point. A modified Solid Isotropic 
Material Penalization material interpolation approach is used in Morphorm, which is defined as 

.. math::
   :label: eq_modified_simp

   z_{min} + (1 + z_{min})z^p

where :math:`z_{min}` is the :ref:`minimum value the density <input_deck_options_scenario_minersatz_kw>`  
can take at a given material point. The :math:`z_{min}` parameter is used to prevent singular matrices 
and thus singular linear system of equations. The parameter :math:`p` denotes a :ref:`penalization 
factor <input_deck_options_scenario_pexp_kw>`, which usually takes on the value of 3. In some applications,
such as stress constrained mass minimization problems, a continuation scheme can be used on the penalization 
factor to aid steer the topology optimization algorithm towards a "0-1" design solution. To avoid numerical 
artifacts that may result from the discretization of the design variables with possibly unstable finite 
element formulations, a :ref:`filter <input_deck_options_method_filter_kws>` is used in most, if not all, 
density-based topology optimization problems. The filter also provides a mechanism to implicitly enforce 
an approximate minimum feature size constraint on the topology optimization problem. While the filter 
does not completely eliminates the issue of mesh-dependencies, it greatly helps control it. The filter
functionality is explained in the next sections. In addition, the upcoming tutorials will cover the steps 
require to set the filter parameters in the Morphorm input deck.

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
candidate material :math:`m`, where :math:`m=1` in :math:numref:`eq_compliance_prob`. 
Therefore, the filtered material points :math:`\hat{z}^m_i` for candidate material :math:`m` 
are given as

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
radius, with respect to material point :math:`x_j^m`. The other type of filter available in 
Morphorm for density-based topology optimization problems is the Helmholtz filter, which will
be covered in a subsequent tutorial. The reader is advice to review the :ref:`filter section <input_deck_options_method_filter_kws>` 
to learn how to best set the kernel filter parameters.  


.. _examples_topt_structTO_density_projection_subsubsec:

Heaviside Projection 
====================

In addition to the use of material interpolation functions and density filters, density-based 
topology optimization problems may also require the use of projection techniques to aid steer 
the optimization algorithm towards a "0-1" design solution. The filter can create transition 
regions with intermediate pseudo-density values. In order to mitigate and avoid the transition 
regions, a projection scheme is employed. The heaviside projection function implemented in 
Morphorm for density-based topology optimization is defined by

.. math::
   :label: eq_proj_func

   \bar{z}^m_j=\frac{\tanh(\beta\eta) + \tanh(\beta(\hat{z}^m_j-\eta))}{\tanh(\beta\eta) + \tanh(\beta(1-\eta))}

where :math:`\eta` governs the density threshold at which the projection takes place and 
:math:`\beta` governs the strength of the projection operation. The :math:`\bar{z}^m_j` are 
the projected material points for candidate material :math:`m`, where :math:`m=1` in this 
tutorial. The parameter :math:`\eta` is set to its default value of 0.5 while a continuation 
scheme is used to update :math:`\beta`. :math:`\beta` can be incrementally increased at a fixed 
frequency to aid steer the optimization algorithm to a "0-1" design solution. The reader is advice 
to review the :ref:`filter section <input_deck_options_method_filter_kws>` to go over the best 
practices on how to set the parameters for the projection scheme. 


.. _examples_topt_structTO_density_fixedblocks_subsubsec:

Fixed Blocks 
************

In topology optimization problems, the optimization algorithm is capable of adding or removing 
material at every material point within the design domain :math:`\Omega`. In some use cases, the 
designer may want to discourage the optimization algorithm from removing material from certain 
regions due to practical engineering considerations. For instance, a component must be mounted on 
top of a surface. Therefore, the optimization algorithm cannot be allow to remove material available 
on this surface as well as other close surrounding areas. The fixed block feature is available in 
Morphorm to enable users to specify non-optimizable regions in the design domain. This information 
is pass to the optimization algorithm and at runtime the algorithm avoids removing material from
the non-optimizable regions. The :ref:`fixed_block_ids <input_deck_options_method_fblocks_ids_kw>` 
parameter in the input deck enables users to specify the element block(s) associated with these 
non-optimizable regions. The fixed block feature will be utilized in the upcoming tutorials to 
show users how to properly set and use the fixed block functionality in their problems. 

.. _examples_topt_compliance_sec:

Compliance Minimization
#######################

The most understood and solved topology optimization problem is complaince minimization. In this 
tutorial, we apply a density-based material description to solve a compliance minimization 
problem. A compliance minimization problem seeks to minimize the structural compliance (maximize 
the stiffness of the structure) given a volume or mass constraint. Mathematically, a compliance 
minimization problem is defined as:

.. math::
   :label: eq_compliance_prob

   \min_{z} \quad & \frac{1}{2}f^T u(\hat{z}) \\
   \textrm{s.t.} \quad & V(\hat{z}) \leq V_{t} \\
   \quad & z_{min}\leq z \leq z_{max} \\ \\
   \textrm{with:} \quad & u(\hat{z})=K^{-1}(\hat{z})f 

Evaluating :math:`u(\hat{z})` requires solving the classic linear elastostatics problem 
:math:`K(\hat{z})u - f = 0`, where :math:`K(\hat{z})` is the stiffness matrix, which depends 
on the :ref:`filtered design variables <examples_topt_structTO_density_filter_subsubsec>` 
:math:`\hat{z}`, and :math:`f` is the force vector. :math:`V_t` is the target volume 
(or mass) while :math:`V(\hat{z})` denotes the current volume of the physical system.  

.. _examples_topt_structTO_density_results_subsubsec:

Results 
*******


.. _examples_topt_stressconst_subsec:

Stress Constrained Mass Minimization
************************************

The consideration of stress constraints in topology optimization formulations is
fundamental to the overall performance of the structure. Stiffness-based designs 
such as compliance minimization problems do not treat material strength as a driving 
design requirement in the topology optimization formulation. Therefore, stiffness-based 
formulations can yield innefective designs due to their inability to meet the stress 
requirements. From a structural engineering standpoint, a more appropriate topology 
optimization formulation should aim to find the lightest structure while not exceeding 
the material strength (stress) requirements at any material point.

Since stress is a fundamental local quantity, a topology optimization formulation 
considering material strenght must incorporate a large number of stress constraints 
to prevent yielding from happening at any material point. The large number of stress 
constraints demands the application of an optimization algorithm capable of managing 
large number of constraints in a computationally efficient manner. Morphorm uses an 
augmented Lagrangian (AL) formulation and high performance computing enabled optimization 
algorithm to effectively solve topology optimization problems with local stress constraints. 
The AL algorithm yields designs that do not violate the stress limit at any material point 
in the design domain while preserving and modeling the local nature of stress. The 
efficiency of the algorithm results from its ability to efficiently handle large number 
of constraints at each optimization step. 

A stress-constrained topology optimization problem aims to find the lightest structure 
capable of supporting the applied loads without experiencing failure at any material 
point in the domain. To limit the yeild stress at points :math:`x_j\in\Omega` stress 
constraints of the form :math:`g_j(u(z),z)\leq{0},\ j=1,\dots,N_p` are imposed, where 
:math:`\Omega` is the design domain and :math:`N_p` denotes the number of material points 
where stress constraints are applied. Thus, in its discrete form, the stress constrained 
mass minimization topology optimization problem is defined as:

.. math::
   :label: eq_stressconst_prob

   \min_{z} \quad & \frac{\mathbf{V}^Tm_V(\hat{z})}{\mathbf{V}^T\mathbf{1}} \\
   \textrm{s.t.} \quad & m_E(\hat{z})\Lambda_j(u(\hat{z}))\left(\Lambda_j^2(u(\hat{z}))+1\right) \leq 0,\quad j=1,\dots,N_p \\
   \quad & z_{min}\leq z \leq z_{max} \\ \\
   \textrm{with:} \quad & \Lambda_j=\frac{\sigma^v_j(u(\hat{z}))}{\sigma_{lim}}-1 \\
   \quad & u(\hat{z})=K^{-1}(\hat{z})f

where :math:`\hat{z}` are the :ref:`filtered designed variables <examples_topt_structTO_density_filter_subsubsec>`,
:math:`\sigma^v_j(u(\hat{z}))` is the Von Mises stress at the j-th material point, and :math:`\sigma_{lim}` is the 
Von mises stress limit at all material points in the domain. The volume interpolation function :math:`m_V(\hat{z})` 
is given by :math:numref:`eq_proj_func` while the Von mises stress interpolation function :math:`m_E(\hat{z})` is 
given by 

.. math::
   :label: eq_vonmises_intrp
   
   z_{min} + (1-z_{min})[m_V(\hat{z})]^p
   
where :math:`m_V(\hat{z})` is given by :math:numref:`eq_proj_func` and :math:`p` is a 
:ref:`penalization factor <examples_topt_structTO_density_subsec>`.