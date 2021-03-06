=================================================================================
Compiler Warning: 'SimpleInjector.Container.InjectProperties(object)' is obsolete
=================================================================================

Starting with Simple Injector 2.6 the **Container.InjectProperties** method will be marked as *obsolete* with the **System.ObsoleteAttribute**. This page describes why we made this decision and identifies things you should consider to mitigate the impact of it's eventual removal from the API.

What's the problem?
===================

The **InjectProperties** method is an unfortunate legacy method that has been part of Simple Injector since version 1. The original intention for **InjectProperties** was to simplify integration scenarios where users were working with technologies (such as Web Forms) that make it impossible to use constructor injection. The **InjectProperties** method performs *implicit property injection*. 

Implicit Property Injection
===========================

Implicit property injection requires the container to inject as many properties as it can, while skipping any properties that it cannot resolve *without warning or exception*. There are many reasons for a property not to be injected: it could be marked internal, have no public setter, have no setter, be static, or its dependencies simply cannot be resolved.
 
With this many reasons to not resolve a dependency any minor misconfiguration in the container or code, for example an accidental change to a property such as making it internal or static, can result in a previously injected property to suddenly be skipped. The outcome of such a change will eventually result in inconsistent and hard to trace bugs or annoying NullReferenceExceptions.

Container configuration errors should be identifiable as :ref:`early in the build <Verify-Configuration>` and execute process as possible and with this aim in mind it is highly desirable to move away from implicit property injection and to focus on *explicit constructor injection* as the primary method for dependency injection. Dependencies should be required most (if not all) of the time, enabling the container to detect misconfigurations in advance and fail fast. Simple Injector does this for you with the *The Simple Injector Pipeline* and the *Diagnostic Services*.

The Simple Injector Pipeline
============================

:doc:`The pipeline <pipeline>` is a series of steps that the Simple Injector container follows when registering and resolving each type. Simple Injector supports customizing certain steps in the pipeline to affect the default behavior. It is important to note that customizations made to this pipeline are *not* reflected in types that are initialized using the now deprecated **InjectProperties** method.

The Diagnostic Services
=======================

Simple Injector 2.0 featured the :doc:`Diagnostic Services <diagnostics>` that analyzes the container's configuration, searching for common configuration mistakes, and giving feedback regarding the configuration at debug time or programmatically through the Diagnostic API (this is especially useful with automated tests). The Diagnostic Services analyze the application's dependency graphs using all of the type information that is available to the container.

For a variety of reasons the Diagnostic Services are unable to detect any runtime calls to the **InjectProperties** method. This makes it impossible for the Diagnostic Services to consider and review these runtime dependencies (these dependencies are invisible to the Diagnostic Services). The **InjectProperties** method allows configuration errors to remain undetected, which can ultimately lead to bugs in your production systems.

For all of these well considered reasons the decision has been made to remove the **InjectProperties** method from the API at some future point in time.

So what do you need to do?
==========================

We advice that you change your class designs and configuration to remove any and all calls to **InjectProperties**. If possible you should consider refactoring your code in such way that property injection is no longer required (i.e. prefer constructor injection), but especially *implicit* property injection. If property injection is still required, the :ref:`IPropertySelectionBehavior extension point <Overriding-Property-Injection-Behavior>` is available that allows enabling property injection in a way that fully integrates with both the pipeline and Diagnostic Services. Please read the :ref:`Advanced Scenarios - Propery Injection <Property-Injection>` documentation page for more information. 

In case **InjectProperties** is used to provide 'build up' behavior, i.e. to let Simple Injector initialize instances that aren't created and managed by Simple Injector, please read the :ref:`Building up External Instances <Building-Up-External-Instances>` section for a better solution.


