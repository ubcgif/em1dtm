.. _overview:

EM1DFM package overview
=======================

Description
-----------

EM1DFM is a Fortran-based, multi-platform program library for carrying out the one-dimensional inversion of frequency-domain, small loop, electromagnetic data acquired to determine the spatial variation of the electrical conductivity and/or magnetic susceptibility of the subsurface. The name "EM1DFM" derives from: electromagnetics ("EM"), one-dimensional models ("1D"), frequency-domain observations ("F"), and magnetic (dipole) sources and
receivers ("M"). 

The acquired data are measurements of the magnetic field due to electric currents and magnetization induced in the subsurface by a time-varying current in a transmitter loop. Information about how the conductivity and susceptibility vary with depth is obtained by making measurements for different frequencies of the transmitter current, and for different separations, heights and orientations of the transmitters and receivers. Different soundings provide information about the lateral variation in the subsurface. A sounding is a distinct collection of FEM measurements (transmitters, receivers and frequencies) which are used to recover a corresponding 1D layered Earth model. On the figure below, we see two examples. On the left, two soundings (e.g. two separate sets of measurements) are collected at the same location; in this case, the inversion should recover the same model. On the right, we see the use of separate soundings to map lateral variation.

.. figure:: ../images/soundings_domain.png
     :align: center
     :width: 700

     Two distinct groupings of transmitters and receivers (soundings) at the same location (left). Different soundings used to map lateral variation in the Earth (right). **Click to enlarge**.


.. figure:: ../images/domain.png
     :align: right
     :figwidth: 50%

     Layered 1D model describing the Earth for each sounding.

For any sounding location, the mathematical representation that EM1DFM uses to model the Earth varies only with depth. In particular, the representation comprises many uniform, horizontal layers, and it is the conductivity and/or susceptibility within each layer that is computed. Four options are available: just the conductivity in each layer can be constructed, or just the susceptibility (constrained to be positive) in each layer, or both the conductivity and susceptibility with the positivity constraint on the susceptibility, or both the conductivity and susceptibility without the positivity constraint. For a complete mathematical treatment of the forward and inversion problem, see :ref:`Background Theory<theory>`.

The initial research underlying this program library was funded principally by the “IMAGE” consortium, of which the following companies were participants: AGIP, Anglo American, Billiton, Cominco, Falconbridge, INCO, MIM, Muskox Minerals, Newmont, Placer Dome and Rio Tinto, and from the Natural Sciences and Engineering Research Council of Canada (NSERC).


Program library content
-----------------------

This package consists of two programs:

   - **EM1DFM:** carries out the inversion of small loop, frequency-domain EM data assuming a layered Earth model

   - **EM1DFMFWD:** a stand-alone program for forward modeling the frequency-domain response assuming a layered Earth model


Licensing
---------

A **constrained educational version** of the program is available with
the `IAG <http://www.flintbox.com/public/project/1605/>`__ package
(please visit `UBC-GIF website <http://gif.eos.ubc.ca>`__ for details).
The educational version is fully functional so that users can learn how
to carry out effective and efficient 3D inversions of magnetic data.
**However, RESEARCH OR COMMERCIAL USE IS NOT POSSIBLE because the
educational version only allows a limited number of data and model
cells**.

Licensing for an unconstrained academic version is available - see the
`Licensing policy document <http://gif.eos.ubc.ca/software/licenses>`__.

**NOTE:** All academic licenses will be **time-limited to one year**.
You can re-apply after that time. This ensures that everyone is using
the most recent versions of codes.

Licensing for commercial use is managed by third party distributors.
Details are in the `Licensing policy document <http://gif.eos.ubc.ca/software/licenses>`__.

Installing
----------

There is no automatic installer currently available for the . Please
follow the following steps in order to use the software:

#. Extract all files provided from the given zip-based archive and place
   them all together in a new folder such as

#. Add this directory as new path to your environment variables.

Two additional notes about installation:

-  Do not store anything in the "bin" directory other than executable
   applications and Graphical User Interface applications (GUIs).

-  A Message Pass Interface (MPI) version is available for Linux upon
   and the installation instructions will accompany the code.





