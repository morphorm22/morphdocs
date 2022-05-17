.. _examples_intro_sec:

Introduction
############

This chapter will introduce the user to the optimization-based design capabilities in
Morphorm through a series of tutorials. Each tutorial covers the all the steps required
to sucessfully run a Morphorm optimization-based design problem. The tutorials will
introduce users to topology and shape optimization concepts and how these concepts
are applied to solve single- and multi-physics optimization-based design problems.
Tutorials on Design of Experiment, Surrogate-Based Optimization, and Pareto Optimization
are presented in this chapter to demonstrate users how to apply these optimization-based
design methods to their problems. Furthermore, this chapter introduces the users to Design
Under Uncertainty methods and explains how these design under uncertainty methods are
applied to produce designs robust to inherent imperfections. A key feature of Morphorm
is its ability to combine multiple optimization-based design methods to build automated
end-to-end design workflows. This chapter will also present several tutorials explaining
how multiple Morphorm optimization-based design methods can be combined to produce an
automated end-to-end design workflow.

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
However, in certain use cases (e.g. structural topology optimization), density-based methods 
may reach an optimized design solution faster than level set methods. 

The tutorials presented in this section describe how to set a topology optimization problem
using density or level set material descriptions. These tutorials will cover a spectrum of 
applications, from the most common single-physics applications to the most complex multi-physics 
applications. Each material description requires additional methods to help steer the optimization 
algorithm towards a design configuration that achieves the best performance given the design 
requirements for :math:numref:`eq_topo_form`. These tutorials will describe how to set and tune 
the parameters associated with these methods based on the choosen material description. Readers 
should be capable of setting both density- or level set based topology optimization problems 
for a plethora of engineering applications after going through the turorials presented herein. 

.. _examples_topt_compliance_subsec:

Structural Topology Optimization
********************************

A structural topology optimization seeks to minimize the structural compliance, i.e. maximize 
the stiffness, given a volume constraint. Mathematically, a structural topology optimization 
problem is defined as:

.. math::
   :label: eq_compliance_prob

   \min_{z} \quad & \frac{1}{2}f^T u(z) \\
   \textrm{s.t.} \quad & V(z) \leq V_{t} \\
   \quad & z_{min}\leq z \leq z_{max}

Evaluating :math:`u(z)` requires solving the classic linear elastostatics problem :math:`K(z)u=f`,
where :math:`K(z)` is the stiffness matrix, which depends on the design variable :math:`z`, and 
:math:`f` is the force vector. :math:`V_t` is the target volume while :math:`V(z)` denotes the 
current volume of the physical system. The volume constraint in :math:numref:`eq_compliance_prob`
can be replaced by a mass constraint if preferred. 

.. _examples_topt_structTO_density_subsubsec:

Density Method
==============

A critical aspect of density-based methods is the proper selection of the material interpolation 
function used to aid steer the optimizer towards a "0-1" design solution. In a density-based topology 
optimization problem, the density values are set to :math:`0\leq{z}_{min}\leq{z}\leq{1}`, where 
:math:`0` denotes the absence of material at a given material point and :math:`1` denotes the 
existence of material at a given material point. A modified Solid Isotropic Material Penalization 
material interpolation function is used in Morphorm, which is defined as 

.. math::
   :label: eq_modified_simp

   z_{min} + (1 + z_{min})*z^p

where :math:`z_{min}` is the :ref:`minimum value the density <input_deck_options_scenario_minersatz_kw>`  
can take to prevent singular matrices and thus a singular linear system of equations. The parameter 
:math:`p` denotes a :ref:`penalization factor <input_deck_options_scenario_pexp_kw>`, which usually 
takes on the value of 3. 

To avoid numerical artifacts that may result from the discretization of the design variables with 
possibly unstable finite element formulations, a :ref:`filter <input_deck_options_method_filter_kws>` 
is applied in most, if not all, density-based topology optimization problems. The filter also offers 
a mechanism to enforce an approximate minimum length scale of features in the optimized design solutions. 
While the filter does not completely eliminates the issue of mesh-dependencies, it greatly helps control 
it. 

.. _examples_topt_structTO_density_filter_subsubsec:

Kernel Filter
-------------

There are two types of filters implemented in Morphorm. The first is the kernel filter, which can take 
on multiple variations. In this :ref:`tutorial <examples_topt_compliance_subsec>`, a linear kernel filter 
is applied to solve :math:numref:`eq_compliance_prob`. The linear kernel filter is defined as

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

   \hat{z}^m_j=\sum_{i=1}^{N_p}=w_{ij}x_i^m
   
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
--------------------

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
to review the :ref:`filter section <input_deck_options_method_filter_kws>` to understand how to 
best set the parameters for the projection scheme, which are set to their default values for 
:ref:`this tutorial <examples_topt_compliance_subsec>`.  
