---
layout: post
title: "Building a XSPICE codemodel outside of the Ngspice source code"
---
The [Ngspice manual](http://ngspice.sourceforge.net/docs/ngspice-31-manual.pdf) (PDF) describes how to build a XSPICE Codemodel within the source code of Ngspice.
In this article I describe how to compile a codemodel only using the installed Ngspice version of your favorite Linux distribution.

## TL;DR

Have a look in the [compile script](https://github.com/der-b/sim_xspice_test/blob/master/compile.sh) of my XSPICE test setting.
In this article I describe what my [test setting](https://github.com/der-b/sim_xspice_test/) is doing.

## Prerequisites

To compile a XSPICE codemodel you need a C compiler and Ngspice.

On Arch Linux you can install both with:

```
$ sudo pacman -S ngspice gcc
```

On Debian/Ubuntu you can install both with:

```
$ sudo apt-get install ngspice gcc
```

## XSPICE Codemodel

Codemodels are loaded by Ngspice as library.
Each library can contain several codemodel.
A codemodel library is created in four steps:

1. Create a interface description of the codemodel.
2. Implement the codemodel.
3. Create the base interface, which Ngspice uses to load the codemodel library.
4. Build the library

The project is located in a directory called *sim_xspice_test*.
All codemodels which are contained in the library are located in a subdirectory under *sim_xspice_test/codemodel*.
During this article a codemodel *clk* is created and will be locaded in *sim_xspice_test/codemodel/clk*.

### Interface Description

The interface description of our codemodel *clk* is located in a file *sim_xspice_test/codemodel/clk/ifspec.ifs* with the following content:

``` 
NAME_TABLE:
C_Function_Name:       cm_clk
Spice_Model_Name:      clk
Description:           "digital clk"

PORT_TABLE:
Port_Name:             out
Description:           "output"
Direction:             out
Default_Type:          d
Allowed_Types:         [d]
Vector:                no
Vector_Bounds:         -
Null_Allowed:          no

PARAMETER_TABLE:
Parameter_Name:        freq
Description:           "Clock Frequency"
Data_Type:             real
Default_Value:         100
Limits:                -
Vector:                no
Vector_Bounds:         -
Null_Allowed:          no

STATIC_VAR_TABLE:
Static_Var_Name:       last_update
Data_Type:             real
Description:           "timestamp of the last output change"
```

The NAME\_TABLE defines the name of the C-Function (*cm_clk*) which implements the codemodel.
The name used by Ngspice to refer to the codemodel in a circuit is *clk*.

The PORT\_TABLE defines the input and output pins of the model we are creating.
In this case, the codemodel has only one digital output which can only be connected to other digital ports (Allowed\_Types).

The PARAMETER\_TABLE defines which parameter can be set by Ngspice.
If the codemodel needs to keep information over consecutive calls to (*cm_clk*), than these variables have to be defined in the STATIC\_VAR\_TABLE.
For more information see section 28.6 of [Ngspice manual](ngspice.sourceforge.net/docs/ngspice-31-manual.pdf) (PDF).

The interface specification have to be translated to a C-File *ifspec.c*, which needs to be compiled into the codemodel library.
This is done by invoking the *cmpp* command:

```
% cd /path/to/sim_xspice_test/
% cd ./codemodel/clk/
% cmpp -ifs
```

### Implementation of the codemodel

The file *sim_xspice_test/codemodel/clk/cfunc.mod* contains the implementation of our codemodel *clk*:

``` c
void cm_clk(ARGS)
{
        Digital_State_t *out, *out_old;
        double time_offset = 1.0/(PARAM(freq)*4);

        if (INIT) { /* initial pass */
                cm_event_alloc(0, sizeof(Digital_State_t));
                out = out_old = (Digital_State_t *) cm_event_get_ptr(0, 0);
                STATIC_VAR(last_update) = TIME;
        } else { /* Retrive previous values */
                out = (Digital_State_t *) cm_event_get_ptr(0, 0);
                out_old = (Digital_State_t *) cm_event_get_ptr(0, 1);
        }

        cm_event_queue(TIME + time_offset);
        if (STATIC_VAR(last_update) + time_offset < TIME) {
                STATIC_VAR(last_update) = TIME;
                *out = ZERO;
                if (*out_old == ZERO) {
                        *out = ONE;
                }
        }

        /*** do things depending on analaysis type ***/
        if (ANALYSIS == DC) { /** DC Analysis **/
                OUTPUT_STATE(out) = *out;
        } else { /** Transient Analysis **/
                if (*out != *out_old) {
                        OUTPUT_STATE(out) = *out;
                        OUTPUT_DELAY(out) = 1e-20;
                        OUTPUT_CHANGED(out) = TRUE;
                } else {
                        OUTPUT_CHANGED(out) = FALSE;
                }
        }
        
        OUTPUT_STRENGTH(out) = STRONG;
}
```

This is a C-Function which uses some XSPICE macros.
Unfortunately, these macros are not C preprocessor macros and need a separate translation step, which can be triggered by the following command:

```
% cd /path/to/sim_xspice_test/
% cd ./codemodel/clk/
% cmpp -mod
```

This creates a C-File *cfunc.c* which also needs to be compiled into the library.

Please note, that the function name have to be the same, as the C\_Function\_Name specified in the interface description.
For more details see section 28.7 in the [Ngspice manual](ngspice.sourceforge.net/docs/ngspice-31-manual.pdf) (PDF) or look into the codemodels predefined by ngspice like the [digital or](https://sourceforge.net/p/ngspice/ngspice/ci/master/tree/src/xspice/icm/digital/d_or/cfunc.mod).

### Base Interface

To generate a base interface two files are needed: *sim_xspice_test/codemodel/modpath.lst* and *sim_xspice_test/codemodel/udnpath.lst*.

The *modpath.lst* contains a list of paths to the codemodels which the library shall include.
In our case, the file only contains one entry for our *clk* module:

```
clk
```

The *udnpath.lst* has to be created, but is empty in our case, since we don't define a "User Defined Node".
See Section 28.8 of the [Ngspice manual](ngspice.sourceforge.net/docs/ngspice-31-manual.pdf) (PDF) for more information.
To generate the base interface, you need to execute the following commands:

```
% cd /path/to/your/project/
% cd ./codemodel/
% cmpp -lst
```

### Build the library

To build the library, a file *dlmain.c* from Ngspice is needed.
The default installation phat for this file is */usr/share/ngspice/*.
The library is build with the following command:

```
% cd /path/to/sim_xspice_test/
% cd ./codemodel/
% cc -I. -fPIC -I./clk/ --shared -lm -lngspice -o clk.cm /usr/share/ngspice/dlmain.c clk/cfunc.c clk/ifspec.c
```

## Testing the code model

To test the codemodel, we create a small circuit which connects the our *clk* codemodel to a predefined codemodel *dac_bridge* which transforms the digital signal to an analog signal.
This signal is then connected to ground through a 1k Ohm pull down resistor.

The circuit is define in a file *sim_xspice_test/test.cir*  :

```
XSpice Test

.model myclk clk (freq=10)

.model dac1 dac_bridge (out_low = 0.0
+                       out_high = 3.3
+                       out_undef = 1.7
+                       input_load = 5.0e-12
+                       t_ris e = 50e-9
+                       t_fall = 20e-9)

aclk dout myclk

abridge1 [dout] [aout] dac1

R aout 0 1k

.tran 10ms 1s

.control
pre_codemodel ./codemodel/clk.cm
run
plot v(aout) v(dout)
.endc

.end
```

The *.control* block tells Ngspice to load the codemodel before parsing the circuit description (*pre\_codemodel*).
Afterwards the circuit is simulated with the command *run* and the results are plotted with the *plot* command.
*v(dout)* is the digital voltage level measured on the connection between our *clk* module and the *dac_bridge*. 

*v(aout)* is the analog voltage level measured on the connection between the *dac_bridge* and the resistor.

The following command will start the simulation:

```
% cd /path/to/sim_xspice_test/
% ngspice ./test.cir
```

The plot, which xspice generates should look like the following screenshot:

[![XSPICE plot](/assets/img/2020/xspice_test_plot.png)](/assets/img/2020/xspice_test_plot.png)

To leave Ngspice, type *quit*.

## Version Information

Used program versions:

```
% ngspice --version
ngspice compiled from ngspice revision 32
Written originally by Berkeley University
Currently maintained by the NGSpice Project

Copyright (C) 1985-1996,  The Regents of the University of California
Copyright (C) 1999-2011,  The NGSpice Project
```

```
% gcc --version
gcc (GCC) 10.1.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

