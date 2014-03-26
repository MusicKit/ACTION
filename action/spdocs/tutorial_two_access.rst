*******************************
Tutorial Two: Access Video Data
*******************************

Abstract
========

This tutorial will demonstrate methods to access, display, and generally use video data gathered through ACTION analysis function.

Prerequisites
=============

* Data analyzed through ACTION, which can take several forms (see below).
* See the first Tutorial for more information.

Raw Data: Files
---------------

The raw analysis is stored in binary files. To summarize, these files, identified by descriptive extensions, are:

#. MOVIE_TITLE.color_lab - Normalized L*a*b* color space: histograms summarizing the distribution of color and luminescence (brightness) over each frame: 8-by-8 grid and full screen.
#. MOVIE_TITLE.opticalflow24 - Optical flow data: motion vectors based on an intermediate corner-detection step: 8-by-8 grid and full screen.
#. MOVIE_TITLE.phasecorr - Phase correlation data: 8-by-8 grid and full screen.
#. MOVIE_TITLE.tvl1 - TV-L1 optical flow data: 8-by-8 grid and full screen.

For a detailed list of the meanings of the various extensions, please see our `Overview <http://bregman.dartmouth.edu/action/action_overview.html>`_. 

Access
======

Import ACTION Classes, Create a Histogram Object, and Play a Movie
------------------------------------------------------------------
We will use color_features_lab for this first example. Let us assume that there are the folowing files: ~/Movies/Vertigo.mov and ~/Movies/Vertigo.hist. Execute the following commands in your Python interpreter:

.. code-block:: python

	from action.suite import *
	vertigo_cfl = ColorFeaturesLAB('Vertigo')
	vertigo_cfl.playback_movie()

You will see the analysis data as bar graphs in the Histogram window. When you have seen enough, press escape to exit the viewer. Below are still shots of the histogram visualization:

.. image:: /images/action_tut2_vertigo_eye.png
.. image:: /images/action_tut2_playback.png


Play segments
-------------
Next try this:

.. code-block:: python

	vertigo_cfl.playback_movie(offset=60, duration=60)

This will play the analysis frames from the second minute. You can also stipulate the offset and duration in frames:

.. code-block:: python

	vertigo_cfl.playback_movie(offset=60, duration=60)

Incidentally, you can find the length of the film by doing the following:
::

	length = vertigo_cfl.determine_movie_length() # in seconds

Raw Data
========
Visualization is nice, however we wish to expose raw data for the user to do something with it. We have a function:

.. code-block:: python

	# data for the central cells, for minute 2
	myseg = Segment(start_time=60, duration=60)
	cfl_data = cfl.all_color_features_for_segment(myseg)	

By default, we return all the features as a tuple (see below for more).

.. code-block:: python

	cfl_data[0].shape
	>>> (240, 48)
	cfl_data[1].shape
	>>> (240, 768)

Segments, simple containers that wrap segments of time, are objects from the Bregman Toolkit for Music Information Retrieval. You may also create them using an ending time rather than duration: `myseg = Segment(start_time=60, end_time=120)`.

Access to Subsets of Histogram Data
-----------------------------------
We provide convenience functions to access histogram data representing different parts of the screen:

* `cfl.full_color_features_for_segment()` will return just the full-screen histogram.
* `cfl.gridded_color_features_for_segment()` will return just the gridded histogram.
* `cfl.middle_band_color_features_for_segment()` will return the cells that represent a band across the center of the screen.
* `cfl.center_quad_color_features_for_segment` will return the cells that represent just the four cells at the center of the screen.
* `cfl.plus_band_color_features_for_segment` will return the cells that represent a plus-shaped "band" (everything but the four corners).

The `for_segment` part of the function means that we will be asking for a (temporal) segment and need to stipulate the onset and duration of such. Here is an example of a full call:
::

	# data for the central cells, for minutes 2-8
	myseg = Segment(120, duration=360)
	cq_data = cfl.center_quad_color_features_for_segment(myseg)

The histogram data will be sized as appropriate. The cell ordering is preserved with gaps eliminated so that bins are numbered from top to bottom, left to right. The data (48 bins of histogram data per cell) is flattened. 
::

	cq_data.shape
	>>> (1440, 192)

1440 = 360 seconds * 4 values per second. 192 = 48 bins per histogram * 4 grid cells that we asked for. We have basically applied a mask that looks like this:
::

	 X  X  X  X
	 X  5  6  X
	 X  9 10  X
	 X  X  X  X

which is indexed in the flattened output array like so:
::

	 X     X       X     X
	 X    0-47   48-95   X
	 X   96-143 144-191  X
	 X     X       X     X

Optical Flow
------------

The same work flow applies to utilizing the optical flow data.

::

	oflow = OpticalFlow('Vertigo')
	myseg = Segment(60, duration=60)
	oflow_data = oflow.middle_band_opticalflow_features_for_segment(myseg)

The optical flow data is collected for all 24 frames in each second, but our access functions have a default stride of 6 frames built in:

.. code-block:: python

	oflow_data.shape
	>>> (240, 256)


Phase Correlation
-----------------

Phase Correlation is identical to OpticalFlow for access...

.. code-block:: python

	pcorr = PhaseCorrelation('Vertigo')
	# etc...


Visualizing the Data as Numpy Arrays
====================================

Finally, let us look at some examples of visualizing this data. Recall that histograms_for_segment will return a tuple: the full screen histogram data and 16 gridded histograms...

.. code-block:: python

	from action.suite import *
	cfl = ColorFeaturesLAB('Vertigo')
	myseg = Segment(60, duration=60)
	mb_data = cfl.middle_band_color_features_for_segment(myseg)
	
	# ACTION has has a function, borrowed from Bregman, which we use here to visualize 2-D arrays
	imagesc(mb_data)

.. image:: /images/action_tut2_mbdata.png

Transpose to see time on the X axis.

.. code-block:: python

	imagesc(mb_data.T)

.. image:: /images/action_tut2_mbdata_T.png

Now look at the all the data from the gridded histgram. You should see that there are now 16 histograms stacked one on top of the other.

.. code-block:: python
	
	gridded_data = cfl.gridded_color_features_for_segment(myseg)
	imagesc(gridded_data.T)

.. image:: /images/action_tut2_gridded_T.png

`Next <tutorial_three_audio.html>`_: a tutorial on audio data and access. After that, we will then do some analysis of the data.