
###########
libpu8
###########

A tiny C++ library for writing portable UTF-8 applications 
##########################################################

*************
Main features
*************

- write your UTF-8 application once, compile it for linux and windows
- translates ``argv`` of the ``main()`` function to UTF-8 if necessary
- make ``std::cin``, ``std::cout`` and ``std::cerr`` work with UTF-8 in all cases. If attached to a file or pipe, UTF-8 is read or written without translation. If attached to a console window in MS windows, the data will be auto-converted from/to UTF-16 such that it is correctly displayed.
- implements two functions ``u8widen`` and ``u8narrow`` (see `<http://utf8everywhere.org/>`_) that convert between UTF-8 and UTF-16 or not, depending on the platform.

*************
Introduction
*************

Motivation
==========

Writing a *portable* C++ unicode program is difficult. In the MS windows world, unicode usually means UTF-16. Windows API functions do in general not accept UTF-8, but instead treat ``char`` as a single character of the user's locale-dependent codepage.

In the linux world, plain ``char`` sequences are almost everywhere interpreted as UTF-8, and UTF-16 is hardly used.


Introductory example
====================

This library enables to write portable code

.. code-block:: cpp

	#include <libpu8.h>
	#include <string>
	#include <fstream>
	#include <windows.h>

	// argv are utf-8 strings when you use main_utf8 instead of main.
	// main_utf8 is a macro. On Windows, it expands to a wmain that calls
	// main_utf8 with converted strings.
	int main_utf8(int argc, char** argv)
	{
		// this will also work on a non-Windows OS that supports utf-8 natively
		std::ofstream f(u8widen(argv[1]));
		if (!f)
		{
			// On Windows, use the "W" functions of the windows-api together 
			// with u8widen and u8narrow
			MessageBoxW(0, 
				u8widen(std::string("Failed to open file ") + argv[1]).c_str(), 0, 0);
			return 1;
		}
		std::string line;
		// line will be utf-8 encoded regardless of whether cin is attached to a 
		// console, or a utf-8 file or pipe.
		std::getline(std::cin, line); 
		// line will be displayed correctly on a console, and will be utf-8 if
		// cout is attached to a file or pipe.
		std::cout << "You said: " << line;
		return 0;
	}

**********
Background
**********

utf8everywhere
==============

The `<http://utf8everywhere.org/>`_ website suggests to use UTF-8 throughout your program, and to use two functions (here: ``u8widen`` and ``u8narrow``) that convert between ``char`` and ``wchar_t`` at the interfaces. On Linux, they are both 8 bit, so u8widen does nothing, and u8narrow is not required. On windows, wchar_t is 16 bit, and the two functions convert between UTF-16 and UTF-8. Furthermore, on windows, ``std::fstream`` has a constructor that accepts a wchar_t UTF-16 string.

Command line arguments (``argv``)
=================================

``argv`` is a different story. This library defines the macro ``main_utf8``. On windows, it expands to a ``wmain`` function body that converts ``argv`` to UTF-8 and calls your ``main_utf8`` function, which is renamed to ``main_utf8_`` by the macro. 


``std::cin``, ``std::cout`` and ``std::cerr``
=============================================

The next problem are the streams ``std::cin``, ``std::cout`` and ``std::cerr``. What is done here was inspired by an answer from StackOverflow. On Linux, the library does nothing. On windows, it is detected if a stream is attached to a console window, or to a file/pipe. Only if attached to a windows console, the data is converted to UTF-16, so that it will get displayed correctly.

***********
Limitations
***********

- C-input-output functions (like scanf, printf) will not work as expected on windows if attached to a console. You need to stick to C++ cin/cout/cerr.
- C functions that expect filenames (such as fopen) will also not work on windows. You need to use C++ fstream.

*********
Building
*********

Building should be straightforward under Linux, or if you are building a Windows console application. When building a Windows non-console application, you need to make sure that the linker calls the ``wmain`` entry function by passing /ENTRY:wmainCRTStartup to the linker.

*********
License
*********

This software was written by Johannes Feulner. It is licensed under the the 3-clause BSD license.

