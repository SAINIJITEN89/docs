VPP Initialization Process
==========================

The Vector Packet Processor (VPP) initialization process can be understood through the following steps:

1. Main function entry point (vlib_unix_main)
---------------------------------------------

The VPP process starts with the ``vlib_unix_main`` function, which can be found in the ``src/vlib/unix/main.c`` file. This function sets up basic process settings and initializes VPP infrastructure components.

.. code-block:: c

   int
   vlib_unix_main (int argc, char *argv[])
   {
     ...
   }

2. Process command-line arguments
---------------------------------

The ``vlib_unix_main`` function initializes the command line parsing library and processes the command-line arguments.

.. code-block:: c

   unformat_init_command_line (&input, (char **) vm->argv);
   if ((e = vlib_plugin_config (vm, &input)))
   {
     clib_error_report (e);
     return 1;
   }
   unformat_free (&input);

3. Plugin early initialization
------------------------------

VPP now performs early plugin initialization by calling ``vlib_plugin_early_init``. This function scans the plugin directory, loads the plugin shared libraries, and calls their initialization functions.

.. code-block:: c

   i = vlib_plugin_early_init (vm);
   if (i)
     return i;

4. Call configuration functions
-------------------------------

VPP calls all configuration functions by invoking ``vlib_call_all_config_functions``. This function processes the configuration for VLIB and VNET components, setting up their respective infrastructure.

.. code-block:: c

   unformat_init_command_line (&input, (char **) vm->argv);
   if (vm->init_functions_called == 0)
     vm->init_functions_called = hash_create (0, /* value bytes */ 0);
   e = vlib_call_all_config_functions (vm, &input, 1 /* early */ );
   if (e != 0)
   {
     clib_error_report (e);
     return 1;
   }
   unformat_free (&input);

5. Initialize ELF main
----------------------

VPP initializes ELF main to load symbols for signal handlers and memory heap backtraces.

.. code-block:: c

   /* always load symbols, for signal handler and mheap memory get/put backtrace */
   clib_elf_main_init (vm->name);

6. Initialize thread stacks
---------------------------

VPP initializes the thread stacks, including the main thread stack.

.. code-block:: c

   vec_validate (vlib_thread_stacks, 0);
   vlib_thread_stack_init (0);

7. Start the main dispatch loop
-------------------------------

Finally, the VPP dispatch loop is started by switching to the main thread's stack and invoking the ``thread0`` function, which eventually calls ``vlib_main_loop_enter``. This function processes the graph nodes and dispatches packets between them.

.. code-block:: c

   __os_thread_index = 0;
   vm->thread_index = 0;

   vlib_process_start_switch_stack (vm, 0);
   i = clib_calljmp (thread0, (uword) vm,
                     (void *) (vlib_thread_stacks[0] +
                               VLIB_THREAD_STACK_SIZE));
   return i;

