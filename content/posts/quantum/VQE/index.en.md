---
weight: 1
title: "What is a Variational Quantum Eigensolver (VQE)"
date: 2020-11-01T16:45:40+08:00
draft: true
author: "Stefano"
description: "VQE"
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["Quantum", "Quantum Machine Learning", "Research", "PhD"]
categories: ["Quantum"]

lightgallery: true
twemoji: false
fraction: true
fontawesome: true
toc:
  enable: true
  auto: true
code:
  copy: true
math:
  enable: true
---

# What is a Variational Quantum Eigensolver (VQE)?

During your journey in Quantum Computing, you may (or better, surely) have stumbled upon what's called Variational Quantum Eigensolver. In this post, I will try to outline the main idea behind this quantum algorithm, which was first introduced by this [paper](https://www.nature.com/articles/ncomms5213?origin=ppub) in 2014.  

A Variational Quantum Eigensolver is a procedure for finding the lowest eigenvalue of a matrix using a quantum computer, and belongs to the wider class of algorithms for quantum systems called Variational Quantum Algorithms (VQA).

This post comes a Jupyter Notebook I prepared fulfilling my application to the [Quantum Open Source Foundation (QOSF)](https://qosf.org/) Mentorship Program. If you don't know what QOSF is, you must definitely go check it out!

In the following, I will give a brief overview of the main ideas form quantum mechanics behind VQE, and then give an explicit example of its implementation using Qiskit, IBM's quantum programming framework. Let's get started! :wink:

## 1 A theoretical minimum

### 1.1 Quantum States
Quantum mechanics deals with the evolution of quantum states, which are mathematically described by vectors belonging to a specific vector space, called _Hilbert Space_. Quantum states are indicated using the so called Dirac _bra-ket_ notation, so that while a generic vector is usually indicated by means of a small arrow on top of the vector, like this $\vec{v}$, a _quantum state_ is denoted as $|\psi\rangle$, which is typical of quantum mechanics. The quantum version of a classical column vector is called "Bra" quantum state and it is indicated $|\cdot\rang$. Similarly, a row vector corresponds to a "Ket" quantum state $\lang \cdot |$, which is equal to the complex conjugate of a bra quantum state:
$$\text{Bra: } |\psi\rang \quad \quad \text{Ket: } \lang \psi | = |\psi\rang ^ \dagger$$
> When we speak of a quantum state $|\psi\rang$, you can picture it as a column vector with some given entries. In the same way, writing $\lang\psi|$ basically means to take the previous vector, transform it to a row vector and then take the complex conjugate of each entry.

At last, the inner (or scalar) product of two quantum states is denoted by the sandwich of a bra with a ket state: $\lang\psi|\phi\rang$.

In a Vector Space, it is possible to consider a _basis_, which is a set of vectors that can be used to express any other vector in that space. Similarly, we can consider a basis of our quantum Hilbert space, denoted as $\lbrace |\phi_i \rangle\rbrace_i$, and use this to express any other quantum state belonging to the space. Thus, for any state in the Hilber Space it will hold:
$$ |\psi\rangle = \sum_{i=1}^N c_i|\phi_i\rangle $$
where $|\psi\rangle$ is a generic quantum state, $c_i \in \mathcal{C}$ are complex coefficients characterizing the quantum state, $N$ is the dimension of the Hilbert Space and $|\phi_i\rangle$ is an element from the basis. In the linear algebra jargon, we say that the state can be expressed as a _linear combination_ of states from a basis.

Operation on quantum states are applied by means of so called Operators, which takes as input a quantum state and sends it to another quantum state. Typically, quantum operations are represented by means of complex unitary matrices. So, for now on, when I speak of an operator, you can think of it as a matrix, in the same way you consider a quantum state as a vector belonging to a very specific vector space.
Let's consider an operator (or matrix) $U$, and a quantum state $\psi$. If it happens that
$$U|\psi\rang = \lambda |\psi\rang$$
that it, the action of the operator leaves the state unchanged apart from a constant factor, we say that that state $\psi$ is an __eigenstate_ of the operator $U$, with _eigenvalue_ $\lambda$. It happens that any Unitary operator (or matrix)
has a specific set of states which are eigenstates of that specific operator. What's more, this set of states actually form a basis of the Hilbert Space, so we can express any quantum state as a linear combination of eigenstates of the considered Operator.   

### 1.2 The Variational Principle

Consider a unitary operator $\mathcal{H}$ whose _eigenstates_ and _eigenvalues_ are $\lbrace |\phi_i\rang \rbrace_i$ and $\lbrace \lambda_i \rbrace_i$ respectively. Since $\mathcal{H}$ is unitary, its eigenvalues are real numbers, and let's suppose that the smallest of them is $\lambda_1$. We call its corresponding eigenstate, $|\phi_1\rang$, the _ground state_ of the operator $\mathcal{H}$.

> The *mean value* of an Operator $\mathcal{H}$ on a given state $|\psi\rang$ is indicated by $\lang \mathcal{H} \rang
 = \lang \psi | \mathcal{H} |\psi \rang$ and consists in applying the operator to the (bra) state, and then considering its inner product with the the original (ket) state.  

Let's now consider the mean value of the operator \mathcal{H} on a generic quantum state $|\psi\rang$:
$$ \lang \mathcal{H} \rang = \lang \psi | \mathcal{H} |\psi \rang $$
expanding the state $|\psi\rang$ in the eigenbasis $\lbrace |\phi_i\rang \rbrace_i$ of the operator, we obtain:
$$ \lang \mathcal{H} \rang  = \sum_{i,j=1}^N c_i c_j^* \lang\phi_j|\mathcal{H}|\phi_i\rang $$
and using the eigenvalue relation $\mathcal{H}|\phi_i\rang = \lambda_i|\phi\rang$
$$ \lang \mathcal{H} \rang  = \sum_{i,j=1}^N c_i c_j^* \lambda_i \lang\phi_j|\phi_i\rang $$
and since states in the basis are orthonormal, that is $\lang\phi_j|\phi_i\rang = 0 \text{ if } i\neq j, \text{ and } 1 \text{ otherwise}$, we eventually obtain:
$$ \lang \mathcal{H} \rang  = \sum_{i=1}^N |c_i|^2\lambda_i $$
First of all, notice that this is a sum of positive terms $|c_i|^2$ multiplied the eigenvalues of $\mathcal{H}$.
If $\psi\rang$ was equal to the ground state of $\mathcal{H}$, that is $\psi\rang=\phi_1\rang$, then it would hold that $\lang \mathcal{H} \rang = |c_1|^2\lambda_1$. In this case the sum reduces to a single term, consisting of a positive number |c_1|^2 multiplied by the lowest of the eigenvalues of $\mathcal{H}$. If however $\psi\rang=\phi_1\rang$.
