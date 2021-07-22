<!--
* Copyright (c) 2017, 2021 IBM Corp. and others
*
* This program and the accompanying materials are made
* available under the terms of the Eclipse Public License 2.0
* which accompanies this distribution and is available at
* https://www.eclipse.org/legal/epl-2.0/ or the Apache
* License, Version 2.0 which accompanies this distribution and
* is available at https://www.apache.org/licenses/LICENSE-2.0.
*
* This Source Code may also be made available under the
* following Secondary Licenses when the conditions for such
* availability set forth in the Eclipse Public License, v. 2.0
* are satisfied: GNU General Public License, version 2 with
* the GNU Classpath Exception [1] and GNU General Public
* License, version 2 with the OpenJDK Assembly Exception [2].
*
* [1] https://www.gnu.org/software/classpath/license.html
* [2] http://openjdk.java.net/legal/assembly-exception.html
*
* SPDX-License-Identifier: EPL-2.0 OR Apache-2.0 OR GPL-2.0 WITH
* Classpath-exception-2.0 OR LicenseRef-GPL-2.0 WITH Assembly-exception
-->

# -Xmint / -Xmaxt


Sets the minimum and maximum proportion of time to spend in the garbage collection (GC) process as a percentage of the overall running time that included the last three GC runs.

- If the percentage of time drops to less than the minimum, the OpenJ9 VM tries to shrink the heap.
- If the percentage of time exceeds the maximum, the VM tries to expand the heap.

:fontawesome-solid-exclamation-triangle:{: .warn aria-hidden="true"} **Restrictions:**

- This option applies only to GC policies that include stop-the-world (STW) operations, such as `-Xgcpolicy:optthruput`.  


## Syntax

| Setting        | Effect                 | Default |
|----------------|------------------------|---------|
|`-Xmint<value>` | Set minimum time in GC | 0.05       |
|`-Xmaxt<value>` | Set maximum time in GC | 0.13      |

For the `gencon` GC policy, the values apply only to the tenure part of the heap. For the `balanced`, `optthruput`, and `optavgpause` GC policies, these values apply to the whole heap. This option cannot be used with the metronome GC policy (`-Xgcpolicy:metronome`) because the heap is always fully expanded.

## See also

- [Heap expansion and contraction](allocation.md#expansion-and-contraction)

<!-- ==== END OF TOPIC ==== xmint.md ==== -->
<!-- ==== END OF TOPIC ==== xmaxt.md ==== -->
