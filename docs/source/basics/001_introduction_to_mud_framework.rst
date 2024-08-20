.. _brief-introduction:

Introduction to mud framework
=============================

From `mud.dev <https://mud.dev/introduction>`_:

*"MUD is a framework for ambitious onchain applications. It reduces the
complexity of building Ethereum apps with a tightly integrated software stack.
It's open source and free to use."*

*"MUD apps are autonomous worlds, infinitely extendable by default. They come
with access control, upgradability, hooks, plugins, and a suite of great
developer tools."*

For smart contract developers
---------------------------------------------

.. important::

  If your dapp doesn't require complexe logic or elaborate data storage, then
  implementing it with native ``Solidity`` would be the best choice. For
  example, a single fungible token ``ERC20`` .

  However, this framework provides you with convenient and friendly data
  reading and writing, system integration and permission control, but it may
  also bring you troubles, such as the most direct one, consuming more gas.

  **So just try and find your own decision.**

When writing a single contract, security is our top priority. While ensuring
correct implementation of business logic, we need to control access rights for
each public interface, such as preventing reentrancy attacks.

Because the Ethereum network limits the size of a smart contract, when our
business logic gradually becomes too complex for a single contract, we begin to
refactor and split the logic into different smart contracts. We even need
additional work to ensure the call permissions between these contracts and data
consistency.

If our project contracts lack upgradability, we would have to redeploy the
entire project contract with each iteration when we are in a rapid iteration
phase. We begin to consider introducing upgradability, separating data and
logic, and allowing upgrades to business logic. At this point, the `Diamond
framework <https://eips.ethereum.org/EIPS/eip-2535>`_ becomes a possible
choice. Dark Forest is a complex on-chain game project built using it. The
Diamond framework allows business logic to be manipulated like facets of a
diamond, including adding, removing, and replacing operations, with all facets
representing business logic sharing a single data storage, ensuring good data
consistency.

But is this the end? Far from it.

Diamond framework is indeed as brilliant as a diamond, a very good and
easy-to-use contract framework. But it's too primitive, requiring polishing of
each facet every time. In other words, the Diamond framework is so generic that
it lacks reusability. It's as if someone tells us a way of thinking to solve a
problem, but we have to think about and implement each step towards the answer
ourselves. So is it possible to have a universal pattern or formula that can be
applied to solve problems?

Mud is like a more advanced contract framework developed on top of the Diamond
framework. While retaining the advantages of the Diamond framework, it
introduces more features familiar to developers, such as plugins, hooks, and
access control. These features make it more convenient for us to build complex
on-chain applications.
