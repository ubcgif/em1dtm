.. _overview:

EM1DTM package overview
=======================

Description
-----------

EM1DTM is a Fortran-based, multi-platform program library for carrying out the
one-dimensional inversion of time-domain, small loop, electromagnetic
data acquired to determine the spatial variation of the electrical
conductivity and/or magnetic susceptibility of the subsurface. The name
"EM1DTM" derives from: electromagnetics ("EM"), one-dimensional models ("1D"),
time-domain observations ("T"), and magnetic (dipole) sources and
receivers ("M").

The observations are values of voltage (i.e., dB/dt) or magnetic ﬁeld.
Receiver coils can be oriented in the x-, y- or z-directions, and they can be
at any position relative to the transmitter loop. The transmitter loop can
have any number of sides (greater than 2), and can be at any height above the
ground surface. It is assumed to be horizontal. The transmitter current
waveform can be a step oﬀ, a linear ramp turn-oﬀ, or a general waveform that
is provided in discretized form. Observations can be for any time after the
step or ramp turn oﬀ, or any time after or during a discretized waveform. All
the observations (in any combination) that are provided for a particular
transmitter loop constitute a “sounding”, and are used to construct the one-
dimensional model for that sounding. Measurement uncertainties can be in the
same units as the observations or as relative uncertainties in percent.


.. figure:: ../images/domain.png
     :align: right
     :figwidth: 50%

     Layered 1D model describing the Earth for each sounding.

The product: an electrical conductivity model. The Earth models are composed of
layers of uniform conductivity with ﬁxed interface depths. The value of the
conductivity in each layer is sought by the inversion. Multiple soundings can
be handled in a single run of the program. Each sounding is inverted
independently for a one-dimensional model under the sounding location, with
the sequence of one-dimensional models written out. These can be viewed
directly as a composite two-dimensional image using the graphical user
interface, or converted to a format which is suitable for viewing as a three-
dimensional image using MeshTools3D.
For any sounding location, the mathematical representation that EM1DTM uses to
model the Earth varies only with depth. In particular, the representation
comprises many uniform "infinite" horizontal layers. For a complete mathematical
treatment of the forward and inversion problem, see :ref:`Background
Theory<theory>`.


General measures for both the measure of data misﬁt and the measure of the amount of model structure:

	- Huber M-measure for data misﬁt, and
	- Ekblom p-measure for model structure, allow for a whole suite of variations, from the traditional sum-
	  of-squares measures, to more robust measures which can ignore outliers in the observations and which
	  can generate piecewise-constant models.



The initial research underlying this program library was funded principally by
the “IMAGE” consortium, of which the following companies were participants:
AGIP, Anglo American, Billiton, Cominco, Falconbridge, INCO, MIM, Muskox
Minerals, Newmont, Placer Dome and Rio Tinto, and from the Natural Sciences
and Engineering Research Council of Canada (NSERC).


Program library content
-----------------------

This package consists of two programs:

   - **EM1DTM:** carries out the inversion of small loop, time-domain EM data assuming a layered Earth model

   - **EM1DTMFWD:** a stand-alone program for forward modeling the time-domain response assuming a layered Earth model


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





