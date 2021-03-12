# Log examples

To help you understand how garbage collection (GC) processes memory for your application and how these processes are recorded, a number of annotated log examples are provided from different GC policies. Each example covers a particular type of cycle from a particular policy. You will learn how to interpret the XML elements in the example log and determine the characteristics of the recorded cycle.

## `gencon` examples

As detailed in [`gencon` policy (default)](gc.md#gencon-policy-default), this policy uses two types of cycle; a partial GC cycle and a global GC cycle. By default, the partial GC cycle runs a *stop-the-world* (STW) *scavenge* operation but can run a concurrent *scavenge* operation ([-Xgc:concurrentScavenge](xgc.md#concurrentscavenge)) on specific platforms if enabled at run time.

The start of a `gencon` cycle is recorded in the log by the following elements and attributes:

<table style="width:100%" align="center">
<caption>Table showing types of <code>gencon</code> cycle along with the corresponding trigger reason and XML elements for each type. </caption>

  <tr>
    <th scope="col">GC cycle</th>
    <th align="center" scope="col">Value of <code>type</code> attribute of the <code>&lt;cycle-start&gt;</code> and <code>&lt;cycle-end&gt;</code>elements</th>
    <th align="center"scope="col">Element that logs the cycle trigger</th>
    <th scope="col">Trigger reason</th>
  </tr>

  <tr>
    <td scope="row"> Global</td>
    <td align="center"><code>global</code> </td>
    <td align="center"><code>&lt;concurrent-kickoff&gt;</code></td>
    <td>Low free memory tenure area threshold reached. Cycle trigger element is located before the <code>&lt;cycle-start&gt;</code> element.</td>
  </tr>

  <tr>
    <td scope="row"> Partial</td>
    <td align="center"> <code>scavenge</code></td>
    <td align="center"><code>&lt;af-start&gt;</code></td>
    <td>Allocation failure. Cycle trigger element is located before the <code>&lt;cycle-start&gt;</code> element.</td>
  <tr>
</table>

You can use the `type` attribute of the `<gc-start>` and `<gc-end>` elements to locate a particular cycle. You can also locate a particular type of cycle by searching for the element that records the cycle trigger, which is located before the `<cycle-start>` element.

You can analyze the increments and operations that are associated with a particular type of cycle by locating and interpreting the elements in the following table:

<table style="width:100%" align="center">
<caption>Table showing increments and operations that are associated with the <code>gencon</code> partial scavenge and global cycles.</caption>

  <tr>
    <th align="center" scope="col">GC process</th>
    <th align="center" scope="col">Elements that log the start and end of the event</th>
    <th align="center" scope="col">Details</th>
  </tr>

  <tr>
    <td align="center" scope="row"> GC cycle</td>
    <td align="center"><code>&lt;cycle-start&gt;</code>, <code>&lt;cycle-end&gt;</code></td>
    <td >The start and end of a GC cycle.</td>
  </tr>

  <tr>
    <td align="center" scope="row"> GC STW increment</td>
    <td align="center"><code>&lt;gc-start&gt;</code>, <code>&lt;gc-end&gt;</code></td>
    <td >The start and end of a GC increment that begins with a pause.</td>
  <tr>
    <tr>
    <td align="center" scope="row"> GC STW increment</td>
    <td align="center"><code>&lt;concurrent-kickoff&gt;</code></td>
    <td>The start of the initial GC increment of the global concurrent cycle that begins the initial mark operation.</td>
  <tr>

  <tr>
    <td align="center" scope="row"> GC STW increment</td>
    <td align="center"><code>&lt;concurrent-global-final&gt;</code></td>
    <td>The start of the final GC increment of the global concurrent cycle that executes the final collection.</td>
  <tr>
  <tr>
    <td align="center" scope="row"> GC operations and suboperations</td>
    <td align="center"><code>&lt;gc-op&gt;</code></td>
    <td>A GC operation such as mark or sweep, or a suboperation such as class unload.</td>
  <tr>
</table>

<i class="fa fa-pencil-square-o" aria-hidden="true"></i> **Note:** For more information about the XML structure of GC cycles, see [GC cycles](vgclog.md#gc-cycles). For more information about GC cycle increments, see [GC increments and interleaving](vgclog.md#gc-increments-and-interleaving).

The following examples use log excerpts to show how the different types of `gencon` cycle are logged.

### Scavenge partial GC cycle

The following example is taken from a `gencon` log. The output is broken down into sections with supporting text to explain the GC processing that is taking place.

To search for a scavenge partial GC cycle, you can search for the `type` attribute value `scavenge` in `cycle-start` and `cycle-end` elements, or search for the `<af>` element that logs the allocation failure trigger.

By default, the `gencon` partial GC cycle runs by using a single *stop-the-world* (STW) pause. The cycle performs only one operation, a *scavenge* operation, which runs only on the *nursery* area. The cycle consists of a single GC increment, which is labeled by using the elements that are shown in the following table:

<table style="width:100%" align="center">
<caption>Table showing the <code>gencon</code> default partial scavenge cycle's GC increment and corresponding XML elements.</caption>

  <tr>
    <th align="center" scope="col">GC operation</th>
    <th align="center" scope="col">GC increment</th>
    <th align="center" scope="col">STW or concurrent</th>
    <th align="center" scope="col">XML element of GC increment</th>
    <th align="center" scope="col">Details</th>
  </tr>
  <tr>
    <td scope="row"> scavenge</td>
    <td align="center">single </code></td>
    <td align="center">STW </code></td>
    <td align="center"><code>&lt;gc-start&gt;</code>, <code>&lt;gc-end&gt;</code></code></td>
    <td>Contains detailed information about copied objects and the weak roots processing operation</code></td>
  </tr>
</table>

The scavenge partial GC cycle follows a general structure in the verbose GC log as shown. Some elements are omitted for clarity:

```xml

<exclusive-start/>          (STW Pause starts)

<af-start/>                 (allocation failure trigger recorded)

<cycle-start/>              (scavenge cycle starts)

<gc-start>                  (scavenge cycle increment starts)

<mem-info>                  (memory status before operation)

<mem></mem>                 (status of different areas of heap)

</mem-info>

</gc-start>

<allocation-stats/>         (snapshot of application threads status taken before cycle started)

<gc-op> “scavenge"</gc-op>  (scavenge operation completed)

<gc-end>                    (scavenge cycle increment ends)

<mem-info>                  (memory status after operation)

<mem></mem>                 (status of different areas of heap)

</mem-info>

</gc-end> 

</cycle-end>                (scavenge cycle ends)

<allocation-satisfied/>     (required allocation has been achieved)

<af-end/>

<exclusive-end>             (STW for scavenge cycle ends)
...

```

The first activity in the cycle is recorded by an `<exclusive-start>` element, which indicates the start of the STW pause. Application (or *mutator*) threads are halted to give the garbage collector exclusive access to the Java&trade; object heap:

```xml
<!-- Start of gencon scavenge partial GC cycle example -->

<exclusive-start id="12392" timestamp="2020-10-18T13:35:45.000" intervalms="406.180">
  <response-info timems="0.070" idlems="0.070" threads="0" lastid="00000000013D6900" lastname="LargeThreadPool-thread-68" />
</exclusive-start>
```

The `<af-start>` element indicates that the cycle was triggered by an allocation failure in the nursery (`type="nursery"`) area of the heap:

```xml
<af-start id="12393" threadId="00000000013D7280" totalBytesRequested="8200" timestamp="2020-10-18T13:35:45.000" intervalms="418.233" type="nursery" />
```

The `<cycle-start>` element marks the start of the cycle. The attribute `type="scavenge"` confirms that this activity is a *scavenge* partial GC cycle:

```xml
<cycle-start id="12394" type="scavenge" contextid="0" timestamp="2020-10-18T13:35:45.000" intervalms="418.231" />
```

Most elements are labeled with an `id` attribute that increases in value incrementally, a`timestamp` attribute, and a `contextid` attribute. All elements that record GC increments and operations that are associated with a particular cycle have a `contextid` value that matches the `id` value of the cycle. The `<cycle-start>` element of this example cycle has an `id="12394"`, so all subsequent elements that have a `contextid="4"`, such as the `<gc-start>` increment element and the `<gc-op>` operation element, are associated with this particular example cycle.  

The `<gc-start>` element records the first GC increment. In this `<gc-start>` section, you can find information about the amount of memory available (`<mem-info>`) and where it is located in the Java&trade; object heap.

The memory snapshot within the `<gc-start>` element is taken before the *scavenge* operation and can be compared with a similar snapshot that is taken afterward to understand the effect on the heap.

```xml
<gc-start id="12395" type="scavenge" contextid="12394" timestamp="2020-10-18T13:35:45.000">
  <mem-info id="12396" free="414960320" total="1073741824" percent="38">
    <mem type="nursery" free="0" total="268435456" percent="0">
      <mem type="allocate" free="0" total="241565696" percent="0" />
      <mem type="survivor" free="0" total="26869760" percent="0" />
    </mem>
    <mem type="tenure" free="414960320" total="805306368" percent="51">
      <mem type="soa" free="374694592" total="765040640" percent="48" />
      <mem type="loa" free="40265728" total="40265728" percent="100" />
    </mem>
    <remembered-set count="21474" />
  </mem-info>
</gc-start>
```

The Java&trade; object heap memory allocation at the start of the increment is as follows:

- The *allocate* space of the nursery area is full, or close to full. The allocation failure was triggered by the lack of available memory in this space.
- The *survivor* space of the nursery area is reported as 'full' to reflect that no available memory is available to allocate to the mutator threads. The entire *survivor* space is reserved for GC operations during the GC increment. 
- The *tenure* area has 395.7 MB (414,960,320B) of free memory available.

The next element `<allocation-stats>` shows a snapshot, which was taken before the cycle started, of the allocation status of the mutator threads. In this example, the thread that used the most memory was `LargeThreadPool-thread-79`.

```xml
<allocation-stats totalBytes="235362176" >
  <allocated-bytes non-tlh="32880" tlh="235329296" />
  <largest-consumer threadName="LargeThreadPool-thread-79" threadId="00000000013F0C00" bytes="6288544" />
</allocation-stats>
```

The *scavenge* GC operation is recorded by the `<gc-op>` element; child elements record details about the operation. For example, 

```xml
<gc-op id="12397" type="scavenge" timems="11.649" contextid="12394" timestamp="2020-10-18T13:35:45.012">
  <scavenger-info tenureage="7" tenuremask="4080" tiltratio="89" />
  <memory-copied type="nursery" objects="154910" bytes="6027440" bytesdiscarded="394832" />
  <memory-copied type="tenure" objects="16171" bytes="562848" bytesdiscarded="3064" />
  <ownableSynchronizers candidates="10838" cleared="10824" />
  <references type="soft" candidates="24" cleared="0" enqueued="0" dynamicThreshold="16" maxThreshold="32" />
  <references type="weak" candidates="390" cleared="269" enqueued="269" />
  <references type="phantom" candidates="1" cleared="0" enqueued="0" />
  <object-monitors candidates="132" cleared="0"  />
</gc-op>
```

The `<memory-copied>` element indicates that 5.75 MB (6,027,440B) of reachable objects were moved by the *scavenge* operation from the allocate space to the survivor space in the nursery area, and 0.54 MB(562,848 B) were moved to the tenure area. 

The `<scavenger-info>` element shows that the [*tenure age*](gc.md#gencon-policy-default) is set to `7`. Any object in the *allocate* space with an age less than or equal to `7` is copied to the survivor space during this `scavenge`operation. Any object that is copied between the *allocate* and *survivor* areas more than `7` times is moved to the tenure area. 

For more information about how the scavenge operation acts on the Java&trade; object heap, see [`gencon` policy (default)](gc.md#gencon-policy-default).

The end of the increment is recorded with `<gc-end>` and provides another snapshot of memory allocation on the heap, similar to `<gc-start>`. 

```xml
<gc-end id="12398" type="scavenge" contextid="12394" durationms="11.785" usertimems="46.278" systemtimems="0.036" stalltimems="0.145" timestamp="2020-10-18T13:35:45.012" activeThreads="4">
  <mem-info id="12399" free="649473560" total="1073741824" percent="60">
    <mem type="nursery" free="235142120" total="268435456" percent="87">
      <mem type="allocate" free="235142120" total="241565696" percent="97" />
      <mem type="survivor" free="0" total="26869760" percent="0" />
    </mem>
    <mem type="tenure" free="414331440" total="805306368" percent="51" macro-fragmented="0">
      <mem type="soa" free="374065712" total="765040640" percent="48" />
      <mem type="loa" free="40265728" total="40265728" percent="100" />
    </mem>
    <pending-finalizers system="0" default="0" reference="269" classloader="0" />
    <remembered-set count="13792" />
  </mem-info>
</gc-end>
```

The Java&trade; object heap memory allocation at the end of the increment is as follows:

- 97% of the *allocate* space of the nursery area is now available as free memory.
- The *survivor* space of the nursery area is still reported as 'full' to reflect that the entire *survivor* space is reserved for GC operations during the next GC increment. 
- The tenure area has 395 MB (414,331,440B) of free memory available. The *scavenge* operation copied 562 KB from the nursery area to the tenure area so less memory is now available in the tenure area.

The *scavenge* operation successfully reclaimed memory in the *allocate* space of the nursery area by copying objects from the *allocate* space into the *survivor* space of the nursery area, and copying objects from the *survivor* space into the tenure area.

The cycle ends (`<cycle-end>`). The following `<allocation-satisfied>` element indicates that the allocation request that caused the allocation failure can now complete successfully. The STW pause ends with the `<exclusive-end>` element:

```xml
<cycle-end id="12400" type="scavenge" contextid="12394" timestamp="2020-10-18T13:35:45.012" />
<allocation-satisfied id="12401" threadId="00000000013D6900" bytesRequested="8200" />
<af-end id="12402" timestamp="2020-10-18T13:35:45.012" threadId="00000000013D7280" success="true" from="nursery"/>
<exclusive-end id="12403" timestamp="2020-10-18T13:35:45.012" durationms="12.319" />

<!-- End of gencon partial GC cycle example -->
```

#### Summary of the example

Analyzing the structure and elements of this example log output shows that this example global cycle has the following characteristics:

- The GC cycle begins with an STW pause due to an allocation failure.

- All GC operations and suboperations that are associated with this cycle occur during the STW pause

- The cycle consists of only 1 GC increment, which runs a single *scavenge* operation.

- The GC cycle reclaims memory in the *allocate* area of the nursery area by coping objects from the *allocate* area to the *survivor* area and also to the tenure area.

### Concurrent scavenge partial GC cycle (non-default)

When concurrent scavenge mode is enabled, the partial GC cycle is run as a [concurrent scavenge cycle](gc.md#concurrent-scavenge). This partial GC cycle is divided into increments to enable the majority of the scavenge operation to run concurrently with running application (or *mutator*) threads. The concurrent increment can run while application threads run, and also while the intermediate concurrent increment of the global GC cycle runs. The interleaving of the concurrent scavenge partial GC cycle with the global cycle can be seen in the logs.

The following elements log the GC increments and operations of the concurrent scavenge partial GC cycle:

<table style="width:100%">
<caption>Table showing the <code>gencon</code> concurrent (non-default) partial scavenge cycle's GC increment and corresponding XML elements.</caption>

  <tr>
    <th align="center" scope="col">GC operation</th>
    <th align="center" scope="col">GC increment</th>
    <th align="center" scope="col">STW or concurrent</th>
    <th align="center" scope="col">XML element of GC increment</th>
    <th align="center" scope="col">Details</th>
  </tr>
  <tr>
    <td scope="row"> scavenge</td>
    <td align="center">initial </code></td>
    <td align="center">STW </code></td>
    <td align="center"><code>&lt;gc-start&gt;</code>, <code>&lt;gc-end&gt;</code></code></td>
    <td>Root scanning, reported as a single scavenge operation.</code></td>
  </tr>
  <tr>
    <td scope="row"> scavenge</td>
    <td align="center">intermediate </code></td>
    <td align="center">concurrent</code></td>
    <td align="center"><code>&lt;concurrent-start&gt;</code>, <code>&lt;concurrent-end&gt;</code></td>
    <td>Live objects are traversed and evacuated (copy-forwarded). Operation is reported as a <code>scavenge</code> operation.</td>
  </tr>
  <tr>
    <td scope="row"> scavenge</td>
    <td align="center">final</code></td>
    <td align="center">STW </code></td>
    <td align="center"><code>&lt;gc-start&gt;</code>, <code>&lt;gc-end&gt;</code></td>
    <td>weak roots scanning, reported as a complex scavenge operation. <code>&lt;gc-op&gt;</code> contains specific details for each of the weak root groups.</code></td>
  </tr>
</table>

To search for a concurrent scavenge partial GC cycle, you can search for the `type` attribute value `scavenge` in `cycle-start` and `cycle-end` elements, or search for the `<af>` element that logs the allocation failure trigger. 

You can locate the concurrent scavenge partial cycle's concurrent increment by searching for `<concurrent-start>` and `<concurrent-end>`. The global cycle's intermediate concurrent increment, which can run at the same time, is not logged by an element, but begins immediately after application threads are restarted following the `<cycle-start type="global"/>` element. For more information about the global cycle's intermediate concurrent increment, see [Log examples - Global GC Cycle](vgclog_examples.md#global-gc-cycle). For more information about GC increments, see [GC increments and interleaving](vgclog.md#gc-increments-and-interleaving).


### `gencon` global GC cycle

The following example shows how a global GC cycle is recorded in a `gencon` policy verbose GC log. The output is broken down into sections with supporting text to explain the GC processing that is taking place.

The global GC cycle runs when the tenure area is close to full, which typically occurs after many partial cycles. As such, the output can be found part way down the full log. For more information about the GC Initialization section, see [Verbose GC log contents and structure ](vgclog.md#verbose-gc-log-contents-and-structure). For an example log output for a `gencon` partial cycle, see [Example - `gencon`’s default partial GC cycle](vgclog.md#example-gencons-default-partial-gc-cycle).

The global GC cycle is split into three increments, as shown in [GC increments and interleaving](./vgclog.md#gc-increments-and-interleaving). Splitting the cycle operations into the following increments reduces pause times by running the majority of the GC work concurrently. The concurrent increment pauses when a partial GC cycle is triggered and resumes after the partial cycle, or multiple cycles, finish. The interleaving of partial GC cycles with the global cycle's intermediate concurrent increment can be seen in the following `gencon` global GC cycle log output. A single partial GC cycle is logged between the initial and final increments of the global cycle.

To search for a global cycle, you can search for the `type` attribute value `global` in `cycle-start` and `cycle-end` elements, or search for the element that logs the initial concurrent increment, `<concurrent-kickoff>`.

 The following elements log the GC increments and operations of the global GC cycle:

 <table style="width:100%">
<caption>Table showing the <code>gencon</code> global cycle's GC increment and corresponding XML elements.</caption>

  <tr>
    <th align="center" scope="col">GC operation</th>
    <th align="center" scope="col">GC increment</th>
    <th align="center" scope="col">STW or concurrent</th>
    <th align="center" scope="col">XML element of GC increment</th>
    <th align="center" scope="col">Details</th>
  </tr>
  <tr>
    <td align="center" scope="row"> n/a - initiates cycle</td>
    <td align="center">initial</td>
    <td align="center">STW</td>
    <td align="center"><code>&lt;concurrent-kickoff&gt;</code></td>
    <td align="center">No <code>&lt;gc-op&gt;</code> is logged. This increment just initiates the concurrent mark increment.</td>
    <td></code></td>
  </tr>
  <tr>
    <td align="center" scope="row"> concurrent mark</td>
    <td align="center">intermediate </code></td>
    <td align="center">concurrent</code></td>
    <td align="center">none</td>
    <td><code>&lt;concurrent-trace-info&gt;</code> records the progress of the concurrent mark increment.</td>
  </tr>
  <tr>
    <td align="center" scope="row"> final collection</td>
    <td align="center">final</code></td>
    <td align="center">STW </code></td>
    <td align="center"><code>&lt;concurrent-global-final&gt;</code></td>
    <td>The increment is typically triggered when a card cleaning threshold is reached. The completion of a tracing phase can also trigger the increment. Operations include a final concurrent mark, a sweep, and an optional class unload and compact.</td>
  </tr>
</table>

The global GC cycle follows a general structure in the verbose GC log as shown. Some child elements are omitted for clarity. Multiple partial GC cycles can start and finish between the start and end of a global GC cycle. In the following example, the structure includes a single partial GC cycle within the global cycle:

```xml

<concurrent-kickoff/>       (global cycle 1st increment recorded)

<exclusive-start/>          (STW pause starts)

<cycle-start/>              (global cycle starts)

<exclusive-end/>            (STW pause ends)

(mutator threads running, global cycle concurrent increment running concurrently)


<exclusive-start/>          (STW for partial GC cycle starts)


...                         (partial GC cycle starts and completes)


<exclusive-end/>            (STW for partial GC cycle ends)

(mutator threads running, global cycle concurrent increment running concurrently)

<exclusive-start/>          (STW pause starts)

<concurrent-global-final/>  (global cycle final increment recorded)

<gc-start/>                 (global cycle final increment starts)

<allocation-stats/>         (snapshot of application threads status taken before cycle started)

<mem-info>                  (memory status before operations)

<mem></mem>                 (status of different areas of heap)

</mem-info>

</gc-start>

<gc-op> “type=rs-scan"</gc-op>        (remembered set scan completed)

<gc-op>”type=card-cleaning" </gc-op>  (card cleaning completed)

<gc-op> “type=mark”</gc-op>           (final mark operation and weak roots processing completed)

<gc-op> “type=classunload”</gc-op>    (class unload operation completed)

<gc-op ”type=sweep” />                (sweep operation completed)

<gc-end>                     (global cycle final increment ends)

<mem-info>                   (memory status after operations)

<mem></mem>                  (status of different areas of heap)

</mem-info>

</gc-end> 

</cycle-end>                 (global cycle ends)

<exclusive-end>              (STW pause ends)

<exclusive-start>            (STW pause starts)
...

```


The first activity in the cycle is recorded by a `<concurrent-kickoff>` element, which records the start of the first of three increments that make up a `gencon` Global GC cycle. The `<concurrent-kickoff>` element records the following information:

- The reason why the GC cycle was triggered. For a `gencon` global cycle, the cycle is triggered when the amount of free memory decreases to a threshold value, the `thresholdFreeBytes` value. 
- The target number of bytes, `targetBytes`, that the cycle aims to mark concurrently.
- The current available memory in the different parts of the heap.

```xml
<concurrent-kickoff id="12362" timestamp="2020-10-18T13:35:44.341">

<kickoff reason="threshold reached" targetBytes="239014924" thresholdFreeBytes="33024922" remainingFree="32933776" tenureFreeBytes="42439200" nurseryFreeBytes="32933776" />

</concurrent-kickoff>
```

For this example, the `remainingFree` bytes value of 31.4 MB (32,933,776B) is approaching the `thresholdFreeBytes` value of 31.5 MB (33,024,922B) so a global cycle is triggered.  

This cycle aims to trace 228 MB (239,014,924B) during the concurrent increment. If the concurrent increment is interrupted by a card cleaning threshold value before it traces all 228 MB, the final STW increment completes the tracing during the STW pause.

<i class="fa fa-pencil-square-o" aria-hidden="true"></i> **Note:** To analyze specific parts of a cycle, you can search for the elements that mark a specific increment of the cycle. For example, you can search for the `<concurrent-global-final>` element to locate the final increment of the `gencon` global cycle. See the details of a particular cycle, such as the [`gencon` global cycle](./vgclog.md/#global-gc-cycle), to determine the element names for particular STW or concurrent GC increments or operations.

The next element recorded in the log, the `<exclusive-start>`element, records the start of an STW pause:

```xml
<exclusive-start id="12363" timestamp="2020-10-18T13:35:44.344" intervalms="342.152">

<response-info timems="0.135" idlems="0.068" threads="3" lastid="00000000015DE600" lastname="LargeThreadPool-thread-24" />

</exclusive-start>
```

The following `<gc-start>` element records details of the start of a new cycle.

```xml
<cycle-start id="12364" type="global" contextid="0" timestamp="2020-10-18T13:35:44.344" intervalms="516655.052" />
```

The `type` attribute records the cycle as a global cycle. The `contextid` of the cycle is, which indicates that all GC events that are associated with this cycle are tagged in relation to the `id` of this cycle. In particular, all subsequent elements that are associated with this particular example cycle have a `contextid` value equal to the `<cycle-start>` `id` attribute value of `“12634”`.

The next element in the log is `<exclusive-end>`, which records the end of the STW pause:

```xml
<exclusive-end id="12365" timestamp="2020-10-18T13:35:44.344" durationms="0.048" />
```

The operations and suboperations of the second increment of the `gencon` global cycle are now running concurrently. 

<!--A blank line appears in the log after this `<exclusive-end>` line and before the next section, which tells you that no STW activities are running. However, concurrent activities might be running. In this case, the concurrent operations  and sub-operations of the second increment of the `gencon` global cycle are running.-->

The next section of the logs records an STW pause that is associated with an allocation failure. The `<cycle-start>` element that follows this STW pause indicates that the cycle is a `scavenge` cycle, which is the partial GC cycle that is used by the `gencon` GC:

```xml
...
<cycle-start id="12368" type="scavenge" contextid="0" timestamp="2020-10-18T13:35:44.582" intervalms="580.047" />
...
```

Subsequent elements have a `contextid=“12368”`, which matches the `id` of this new `scavenge`cycle. For more information about how this cycle is recorded in the logs, see [Scavenge partial GC cycle](#scavenge-partial-gc-cycle).


<!---
```xml
<exclusive-start id="12366" timestamp="2020-10-18T13:35:44.582" intervalms="237.874">

<response-info timems="0.094" idlems="0.033" threads="5" lastid="00000000014E0F00" lastname="LargeThreadPool-thread-67" />

</exclusive-start>

...

<af-start id="12367" threadId="00000000013D7280" totalBytesRequested="96" timestamp="2020-10-18T13:35:44.582" intervalms="580.045" type="nursery" />

<cycle-start id="12368" type="scavenge" contextid="0" timestamp="2020-10-18T13:35:44.582" intervalms="580.047" />

...

<exclusive-end id="12377" timestamp="2020-10-18T13:35:44.594" durationms="11.816" />
```
--->

The operations and suboperations of the second, concurrent increment of the `gencon` global cycle are paused while the STW `scavenge` operation is running, and resume when the STW pause finishes. 

After the partial GC cycle completes and the STW pause finishes, the log records a new STW pause, which is triggered to enable the final `gencon` global GC increment to run. This final increment finishes marking the nursery area and completes the global cycle. The `<exclusive-start>` element is followed by a `<concurrent-global-final>` element, which logs the beginning of this final increment (and by implication, the end of the second increment).

```xml
<exclusive-start id="12378" timestamp="2020-10-18T13:35:44.594" intervalms="12.075">

<response-info timems="0.108" idlems="0.040" threads="3" lastid="00000000018D3800" lastname="LargeThreadPool-thread-33" />

</exclusive-start>

<concurrent-global-final id="12379" timestamp="2020-10-18T13:35:44.594" intervalms="516905.029" >

<concurrent-trace-info reason="card cleaning threshold reached" tracedByMutators="200087048" tracedByHelpers="12164180" cardsCleaned="4966" workStackOverflowCount="0" />

</concurrent-global-final>
```

The `reason` attribute of the `<concurrent-trace-info>` child element indicates that this final STW increment of the global cycle was triggered because a card-cleaning threshold was reached. The concurrent tracing was stopped prematurely and the `targetBytes` concurrent tracing target, recorded at the cycle start by `<concurrent-kickoff>`, was not achieved concurrently. If the concurrent tracing completes without interruption, the `<concurrent-trace-info` element logs `reason=tracing completed`.  

In the next section that begins with the `gc-start` element, you can find information about the amount of memory available (`<mem-info>`) and where it is located in the java&trade; object heap. This snapshot is taken before the final increment's operations and suboperations are run and can be compared with a similar snapshot that is taken afterward to understand the effect on the heap. The child element attribute values of the`<mem>` and `<mem-info>` elements indicate the status of the memory.

<i class="fa fa-pencil-square-o" aria-hidden="true"></i> **Note:** You can double check that the increment is associated with the GC global cycle in the example by checking the `contextid` attribute value matches the `id=12364` attribute value of the cycle's <gc-cycle> element.

```xml
<gc-start id="12380" type="global" contextid="12364" timestamp="2020-10-18T13:35:44.594">

<mem-info id="12381" free="277048640" total="1073741824" percent="25">

<mem type="nursery" free="234609440" total="268435456" percent="87">

<mem type="allocate" free="234609440" total="241565696" percent="97" />

<mem type="survivor" free="0" total="26869760" percent="0" />

</mem>

<mem type="tenure" free="42439200" total="805306368" percent="5">

<mem type="soa" free="2173472" total="765040640" percent="0" />

<mem type="loa" free="40265728" total="40265728" percent="100" />

</mem>

<pending-finalizers system="0" default="0" reference="405" classloader="0" />

<remembered-set count="17388" />

</mem-info>

</gc-start>

<allocation-stats totalBytes="827488" >

<allocated-bytes non-tlh="96" tlh="827392" />

<largest-consumer threadName="LargeThreadPool-thread-68" threadId="00000000013D6900" bytes="65632" />

</allocation-stats>
```

The element `<allocation-stats>` shows a snapshot, which was taken before the cycle started, of the allocation status of the mutator threads. In this example, the thread that used the most memory was `LargeThreadPool-thread-68`.

For this example, at the start of this GC increment, the tenure area is low on free memory, as expected. 25% of the total heap is available as free memory, which is split between the following areas of the heap:

- The nursery area, which has 223.7 MB (234,609,440B) of free memory available. The free memory is only available in the *allocate* space of the nursery area. The *survivor* space of the nursery area is reported as 'full' to reflect that no available memory is available to allocate to the mutator threads. The entire *survivor* space is reserved for GC operations during the GC increment. 
- The tenure area, which has 40.5 MB (42,439,200B) available as free memory, which is only 5% of its total memory. Most of this free memory is in the large object area (LOA). Almost no free memory is available in the small object area (SOA). 

The `<gc-op>` elements and their child elements contain information about the increment’s operations and suboperations. The final increment of the `gencon` global cycle consists of multiple operations, each logged with a `<gc-op>` element. The type of operation is shown by the `<gc-op>` `type` attribute. The final increment of the example log runs five types of operation:

1. `Rs-scan`
2. `Card-cleaning`
3. `Mark`
4. `Classunload`
5. `Sweep`

<i class="fa fa-pencil-square-o" aria-hidden="true"></i> **Note:** The final increment of a `gencon` global cycle can include an optional compact suboperation. 

<i class="fa fa-pencil-square-o" aria-hidden="true"></i> **Note:** For more information about the different types of GC operation, see [GC operations](gc_overview.md#gc-operations).

```xml
<gc-op id="12382" type="rs-scan" timems="3.525" contextid="12364" timestamp="2020-10-18T13:35:44.598">

<scan objectsFound="11895" bytesTraced="5537600" workStackOverflowCount="0" />

</gc-op>

<gc-op id="12383" type="card-cleaning" timems="2.910" contextid="12364" timestamp="2020-10-18T13:35:44.601">

<card-cleaning cardsCleaned="3603" bytesTraced="5808348" workStackOverflowCount="0" />

</gc-op>

<gc-op id="12384" type="mark" timems="6.495" contextid="12364" timestamp="2020-10-18T13:35:44.607">

<trace-info objectcount="1936" scancount="1698" scanbytes="61200" />

<finalization candidates="389" enqueued="1" />

<ownableSynchronizers candidates="5076" cleared="523" />

<references type="soft" candidates="18420" cleared="0" enqueued="0" dynamicThreshold="32" maxThreshold="32" />

<references type="weak" candidates="19920" cleared="114" enqueued="60" />

<references type="phantom" candidates="671" cleared="50" enqueued="50" />

<stringconstants candidates="40956" cleared="109" />

<object-monitors candidates="182" cleared="51" />

</gc-op>

<gc-op id="12385" type="classunload" timems="1.607" contextid="12364" timestamp="2020-10-18T13:35:44.609">

<classunload-info classloadercandidates="425" classloadersunloaded="6" classesunloaded="2" anonymousclassesunloaded="1" quiescems="0.000" setupms="1.581" scanms="0.019" postms="0.007" />

</gc-op>

<gc-op id="12386" type="sweep" timems="9.464" contextid="12364" timestamp="2020-10-18T13:35:44.618" />
```

The end of the increment is recorded with `<gc-end>` and provides another snapshot of memory in the heap, similar to `<gc-start>`. 


```xml
<gc-end id="12387" type="global" contextid="12364" durationms="24.220" usertimems="86.465" systemtimems="0.000" stalltimems="2.846" timestamp="2020-10-18T13:35:44.618" activeThreads="4">

<mem-info id="12388" free="650476504" total="1073741824" percent="60">

<mem type="nursery" free="235516088" total="268435456" percent="87">

<mem type="allocate" free="235516088" total="241565696" percent="97" />

<mem type="survivor" free="0" total="26869760" percent="0" />

</mem>

<mem type="tenure" free="414960416" total="805306368" percent="51" micro-fragmented="98245682" macro-fragmented="0">

<mem type="soa" free="374694688" total="765040640" percent="48" />

<mem type="loa" free="40265728" total="40265728" percent="100" />

</mem>

<pending-finalizers system="1" default="0" reference="515" classloader="0" />

<remembered-set count="13554" />

</mem-info>

</gc-end>
```

60% of the heap now contains free memory as a result of the final global cycle increment, which is split between the following areas of the heap:

- The nursery area, which gained 0.9 MB of free memory. The nursery area now has 224.6 MB (235,516,088B) available as free memory. At the start of the final increment, the nursery area had 223.7 MB (234,609,440B) of free memory available.
- The tenure area, which gained 355.2 MB (372,521,216B) of free memory. (the tenure area now has 395.7 MB (414,960,416B) available as free memory. At the start of the final increment, the tenure area had 40.5 MB (42,439,200B) of free memory available).

<i class="fa fa-pencil-square-o" aria-hidden="true"></i> **Note:** The global GC cycle runs to reclaim memory in the tenure area. The freeing up of memory in the nursery area is achieved by using the partial GC cycle. For more information, see [`gencon` policy (default)](gc.md#gencon-policy-default).

After the final increment of the global cycle completes, the global cycle ends and the STW pause ends, as shown in the following output:

```xml
<cycle-end id="12389" type="global" contextid="12364" timestamp="2020-10-18T13:35:44.619" />

<exclusive-end id="12391" timestamp="2020-10-18T13:35:44.619" durationms="24.679" />
```

#### Summary of the example

Analyzing the structure and elements of this example log output shows that this example global cycle has the following characteristics:

- The GC global cycle is triggered when a memory threshold is reached and begins with an STW pause.

- After the first increment of the GC global cycle completes, the STW pause ends and the second increment runs concurrently.

- A single partial GC cycle starts and finishes between the start and end of the concurrent increment.

- An STW pause begins after the concurrent increments completes, during which the third and final increment of the global cycle, which consists of five operations, runs.

- The global GC cycle reclaims memory in the tenure area and a small amount of memory in the nursery area.

## `balanced` examples

The [`balanced`](gc.md#balanced-policy) policy (`-Xgcpolicy:balanced`) uses two types of cycle to perform GC – a partial GC cycle and a global GC *mark* cycle. The `balanced` policy GC may have to perform a third type of cycle - a global cycle - to reclaim memory after an allocation failure that has resulted from tight memory conditions.
 
Note: For more information about the cycles used in a particular policy, see [garbage collection policies](gc#garbage-collection-policies).

The start of a `balanced` cycle is recorded in the log by the following elements and attributes:

| GC cycle or increment | value of `type` attribute of the `<cycle-start>` and `<cycle-end` elements| Element that logs the cycle or increment trigger| Triggering reason|
|----------|----------------------------|------------------------------|------------------|
|Global mark cycle| `global mark phase`| `<allocation-taxation>` | taxation threshold reached. |
| Global mark increment of global mark cycle| `GMP work packet processing` | `<concurrent-start>` `<concurrent-end>` |
partial cycle| `partial gc`           | `<allocation-taxation>`                 |taxation threshold reached|
|global cycle|     `global garbage collect`    | `<sys-start reason="explicit"/>` |An allocation failure. Occurs under tight memory conditions. Cycle runs rarely. |

To locate a particular type of cycle, you can search for the `type` attribute of the `<cycle-start>` and `<cycle-end>` elements.

When memory in the Java&trade; object heap reaches a memory threshold called an allocation taxation threshold,  a GC cycle or GC increment is triggered. Partial GC cycles, global mark cycles, and global cycles set the allocation taxation threshold at the end of their cycle. For `balanced` cycles, the *taxation* on the mutator threads refers to pausing the mutator threads while GC work is run.

If a partial GC cycle is not run between global mark phase increments of a global *mark* cycle, the allocation taxation threshold is set to trigger the next cycle when the *eden* space is full. Specifically, at the end of the partial gc cycle, the allocation threshold is set to be equal to the size of the *eden* space.

If a partial GC cycle is run within a global *mark* cycle, the allocation taxation threshold is set to be smaller than the size of the *eden* space to allow for global mark phase increments to run in between partial GC cycles. Specifically, during the global mark cycle, the allocation taxation threshold is set to be half the size of the eden space.

Note: For more information about GC increments, see [GC increments and interleaving](vglog.md#gc-increments-and-interleaving).

You can analyze the increments and operations that are associated with a particular type of cycle by locating and interpreting the elements in the following table:

| GC process             | start and end XML elements   | Details |
|------------------------|------------------------------|------------------------------|
|GC cycle                |`<cycle-start>`, `<cycle-end>`| The start and end of a GC cycle.|
|GC STW increment        |`<gc-start>`, `<gc-end>`      | The start and end of a GC increment or sub-increment that begins with a *stop-the-world* pause. For example a `global mark phase` global mark GC cycle increment or a partial GC cycle increment.
|GC concurrent increment        | `<concurrent-start>`, `<concurrent-end>` | The start of the concurrent *global mark phase work packet processing* sub-increments of the global mark cycle|
|GC operations and phases| `<gc-op>`                    | A GC operation such as mark or sweep, or a suboperation ‘phase’ such as class unload. |

Note: For more information about the XML structure of GC cycles, see [GC cycles](vgclog.md#gc-cycles).

The following sections use log excerpts to show how the different types of give details about `balanced` cycle are logged. 

### `balanced` partial GC cycle

The following [`balanced` partial GC cycle](gc.md#balanced-policy) example is taken from a `balanced` policy verbose GC log. The output is broken down into sections to explain the GC processing that is taking place.

To search for a `balanced` partial GC cycle, you can search for the `type` attribute value `partial gc` in `<cycle-start>` and `<cycle-end>` elements.

The `balanced` policy’s partial GC cycle reclaims memory in the heap for allocation of new objects by reducing the number of used regions. The partial GC cycle always reduces used regions in the *eden* space and may also reclaim memory from older regions. Multiple partial GC cycles often run in between global mark phase increments of the [`balanced` global mark cycle](vgclog_examples.md#balanced-global-mark-cycle).  

All the partial GC cycle's operations run during a single *stop-the-world* pause:

|GC Operations | GC increment | *stop-the-world* or concurrent| XML element of GC increment          | Details                                                                   |
|-------------|--------------|-------------------------------|--------------------------------------|---------------------------------------------------------------------------|
|copy forward, and optionally class unload, sweep, and compact |Single        | *stop-the-world*              | `<gc-start>`, `<gc-end>`| `<gc-op'>` |ADD| 

The `balanced` partial GC cycle follows a general structure in the verbose GC log as shown. The lines are indented to help illustrate the flow and some child elements are omitted for clarity:

```xml
<exclusive-start/>                       (STW pause starts)

<allocation-taxation/>                   (memory threshold trigger recorded)

<cycle-start/>                            (partial cycle starts)

<gc-start/>                             (partial  cycle increment starts)

<mem-info>                            (memory status before operations)

 <mem></mem>                         (status of different types of memory)

</mem-info>         

</gc-start> 

<allocation-stats>                      (snapshot of application threads status...
                                          ... taken before the cycle starts)
<gc-op> type="copy forward" </gc-op>    (copy forward operation completed)

<gc-op> type="class unload" </gc-op>    (class unload operation completed)

<gc-op> type="sweep" </gc-op>           (sweep operation completed)

<gc-op> type="compact" </gc-op>         (compact operation completed)

<gc-end>                                (partial cycle increment ends)

<mem-info>                            (memory status after operations)
          
<mem></mem>                         (status of different types of memory)

</mem-info>         

</gc-end>

<cycle-end>                               (partial cycle ends)

<exclusive-end>                           (STW pause ends)
```

When the `balanced` partial GC cycle is triggered, the GC runs an STW pause. Application (or *mutator*) threads are halted to give the garbage collector exclusive access to the heap. The STW pause is recorded in the logs by the `<exclusive-start>` element. 

```xml
<exclusive-start id="184" timestamp="2021-02-26T11:11:42.310" intervalms="3745.790">
  <response-info timems="3.138" idlems="1.056" threads="2" lastid="00000000006EDE00" lastname="RunDataWriter.1" />
</exclusive-start>
```

A `balanced` partial GC cycle is triggered when the region count for the *eden* space reaches the allocation taxation threshold. The logs record this trigger reason by using the`<allocation-taxation>` element.

```xml
<allocation-taxation id="185" taxation-threshold="2147483648" timestamp="2021-02-26T11:11:42.311" intervalms="3745.785" />
```

Details about the start of the cycle are recorded by the `<cycle-start>` element. The cycle is recorded as a `partial gc` with an `id=336`. Any subsequent elements associated with this cycle have a `contextid=186` to match the cycle's `id`. You can use this `contextid` value to distinguish the partial GC cycle increment and operations from interleaving increments and operations from other `balanced` cycles, such as global *mark* cycles.

```xml
<cycle-start id="186" type="partial gc" contextid="0" timestamp="2021-02-26T11:11:42.311" intervalms="3745.805" />

```

The partial cycle begins its one and only GC increment, recorded using the `<gc-start>` element. You can understand the effect that the increment operations have on the heap by comparing snapshots of the memory taken at the start and the end of the increment.  The child elements `<mem-info>` and `<mem>` of the `<gc-start>` and `<gc-end>` elements record the amount of memory available and where it is located in the heap.

```xml
<gc-start id="187" type="partial gc" contextid="186" timestamp="2021-02-26T11:11:42.311">
  <mem-info id="188" free="897581056" total="4294967296" percent="20">
    <mem type="eden" free="0" total="2147483648" percent="0" />
    <arraylet-primitive objects="1" leaves="4" largest="4" />
    <remembered-set count="2749664" freebytes="160705664" totalbytes="171704320" percent="93" regionsoverflowed="1" regionsstable="12" regionsrebuilding="0"/>
  </mem-info>
</gc-start>
```

As expected, at the start of this increment, the eden region is full. 856MB (897,581,056 B) of the total 4096MB (4294,967,296 B) heap, equivalent to 20% of the heap, is available as free memory.

The status of the *remembered set*, a metastructure specific to openJ9 JVM generational garbage collectors, is reported by the `<remembered-set>` element. The remembered set metastructure keeps track of any object references that cross different regions. Each region corresponds to a single remembered set. 

The partial gc cycle uses and prunes the remembered set. The `regionsoverflowed` value records the number of regions that exceeded the non-object-heap memory allocation that is reserved for the remembered set. The partial GC cycle cannot reclaim memory from these overflow regions. The partial GC cycle also cannot reclaim memory from any regions whose remembered set is being rebuilt by an increment of a global mark cycle that is in progress.

At the start of the partial gc cycle, the remembered set is using 93% of its available memory capacity, with 153.26 MB (160705664 B) available. The set consists of 2,749,664 cards and has 1 overflow region.

The following element, `<allocation-stats>`, records information about the status of the application (or *mutator*) threads before the start of the current cycle. For this example, the thread `Group1.Backend.CompositeBackend{Tier1}.7` was the largest consumer of memory.

```xml
<allocation-stats totalBytes="2146431360" >
  <allocated-bytes non-tlh="96417448" tlh="2050013912" arrayletleaf="0"/>
  <largest-consumer threadName="Group1.Backend.CompositeBackend{Tier1}.7" threadId="00000000007E9300" bytes="275750048" />
</allocation-stats>
```

The operations of the GC increment are run and details are recorded in the `<gc-op> elements`. The logs show that this increment begins with a copy forward operation followed by a class unload. Other `balanced` partial GC cycles can also include sweep and compact operations. For details about the operations involved in `balanced` partial GC cycles, see [Balanced Policy - GC Processing](gc.md#balanced-policy). 

```xml
<gc-op id="189" type="copy forward" timems="400.637" contextid="186" timestamp="2021-02-26T11:11:42.713">
  <memory-copied type="eden" objects="4434622" bytes="119281928" bytesdiscarded="1382272" />
  <memory-copied type="other" objects="8847813" bytes="244414264" bytesdiscarded="6243176" />
  <memory-cardclean objects="1446970" bytes="64143048" />
  <regions eden="512" other="80" />
  <remembered-set-cleared processed="2435794" cleared="887129" durationms="8.667" />
  <finalization candidates="66" enqueued="56" />
  <ownableSynchronizers candidates="256500" cleared="78012" />
  <references type="soft" candidates="153648" cleared="0" enqueued="0" dynamicThreshold="22" maxThreshold="32" />
  <references type="weak" candidates="1266" cleared="610" enqueued="430" />
  <stringconstants candidates="9479" cleared="0"  />
  <object-monitors candidates="13576" cleared="13505"  />
</gc-op>
<gc-op id="190" type="classunload" timems="0.010" contextid="186" timestamp="2021-02-26T11:11:42.713">
  <classunload-info classloadercandidates="179" classloadersunloaded="0" classesunloaded="0" anonymousclassesunloaded="0" quiescems="0.000" setupms="0.010" scanms="0.000" postms="0.000" />
</gc-op>
```

The logs show that the copy forward operation acts on the the entire *eden* space (512 regions), recorded as `type=eden`, and 80 older regions, which are recorded as `type=other`. 113.76 MB (119281928 B) have been copied from the *eden* space to 1st generation regions and 233.10 MB (244414264 B) of memory in non-*eden* regions have been copied to the next generation of regions. The copy forward operation is followed by a class unload operation.

In some cases, a copy forward operation moves some regions by copying forward the objects in those regions, but only marks the objects in other regions. For example, the following log excerpt is taken from  a different partial cycle, corresponding to a `contextid` of `2049`. The copy forward operation in the following example involves marking some regions and copying forward others.

```xml
<gc-op id="2052" type="copy forward" timems="649.059" contextid="2049" timestamp="2021-02-26T11:22:34.901">
  <memory-copied type="eden" objects="95989" bytes="7882704" bytesdiscarded="501088" />
  <memory-copied type="other" objects="2955854" bytes="86854064" bytesdiscarded="626024" />
  <memory-cardclean objects="1304" bytes="56840" />
  <memory-traced type="eden" objects="23392785" bytes="553756840" />
  <memory-traced type="other" objects="5461302" bytes="131394216" />
  <regions eden="488" other="138" />
  <remembered-set-cleared processed="156775" cleared="4897" durationms="1.759" />
  <finalization candidates="31" enqueued="12" />
  <ownableSynchronizers candidates="1992467" cleared="1600904" />
  <references type="soft" candidates="329190" cleared="0" enqueued="0" dynamicThreshold="8" maxThreshold="32" />
  <references type="weak" candidates="697" cleared="105" enqueued="6" />
  <stringconstants candidates="9848" cleared="0"  />
  <object-monitors candidates="1437" cleared="1353"  />
  <heap-resize type="expand" space="default" amount="0" count="1" timems="0.000" reason="continue current collection" />
  <warning details="operation aborted due to insufficient free space" />
</gc-op> 
```

The logs record these 2 concurrent parts of a copy-forward operation in the `<gc-op type="copy forward">` section by using a `<memory-traced>` child element. In addition,`evacuated` and `marked` attributes for the `<regions>` child element are used to distinguish between the number of regions that have been copied forward (recorded as `evacuated`) and the number of regions that have just been marked. For example, `<regions eden="256" other="308" evacuated="308" marked="256" />`.

Returning to our `contextid=186` partial cycle example, the next element in the logs, `<gc-end>`, records the end of the increment and provides another snapshot of memory allocation on the heap, similar to `<gc-start>`.

```xml
<gc-end id="191" type="partial gc" contextid="186" durationms="402.645" usertimems="3157.520" systemtimems="4.000" stalltimems="47.689" timestamp="2021-02-26T11:11:42.714" activeThreads="8">
  <mem-info id="192" free="3003121664" total="4294967296" percent="69">
    <mem type="eden" free="2147483648" total="2147483648" percent="100" />
    <arraylet-primitive objects="1" leaves="4" largest="4" />
    <pending-finalizers system="56" default="0" reference="430" classloader="0" />
    <remembered-set count="2922048" freebytes="160016128" totalbytes="171704320" percent="93" regionsoverflowed="1" regionsstable="12" regionsrebuilding="0"/>
  </mem-info>
</gc-end>

```

The heap memory allocation at the end of the increment is as follows:

- The heap now has 2864 MB(3,003,121,664 B) of memory available compared to the 856 MB available at the start of the increment. The increment reclaimed 2,008 MB of memory in the heap, which is slightly less than the size of the eden space, as expected.

- The eden space is recorded to have 100% memory available as free memory. (The eden space, which consists of regions containing the newest objects allocated, has been fully recreated by reclaiming almost all of the eden regions - some objects from eden regions always survive - and assigning some other empty regions of the heap to the eden space).

- The remembered set count has increased by 172,384 cards, and the number of freebytes in the remembered set has decreased by 0.66 MB (689,536 B).

The cycle completes and the GC restarts application threads.

```xml
<cycle-end id="193" type="partial gc" contextid="186" timestamp="2021-02-26T11:11:42.714" />
<exclusive-end id="194" timestamp="2021-02-26T11:11:42.714" durationms="404.145" />
```

The next cycle recorded in the logs is another partial GC cycle. The `<gc-start>`element records the following information:

```xml
<gc-start id="198" type="partial gc" contextid="197" timestamp="2021-02-26T11:11:46.072">
  <mem-info id="199" free="855638016" total="4294967296" percent="19">
    <mem type="eden" free="0" total="2147483648" percent="0" />
    <arraylet-primitive objects="1" leaves="4" largest="4" />
    <remembered-set count="2922048" freebytes="160016128" totalbytes="171704320" percent="93" regionsoverflowed="1" regionsstable="12" regionsrebuilding="0"/>
  </mem-info>
</gc-start>
```

The `<mem-info>` element shows that the following occurred in between the end of the last (partial) GC cycle and the start of this one:

- All available memory in the *eden* area was allocated to application threads.
- Application threads also used some memory from non-*eden* heap areas. The heap's total available memory reduced from the 69% to 19%.
- The `<remembered-set` element shows that the remembered set status is unchanged. When mutator threads run, they build data about object references that cross boundaries using a card table. However, processing of card table data into the remembered set and reporting of the remembered set counts are executed during the operations of a cycle.

#### Summary of the example

Analyzing the structure and elements of this example log output shows that this example `balanced` partial GC cycle has the following characteristics:

- The partial GC cycle begins with an STW pause and is triggered because a memory allocation threshold was reached. The *eden* space is full.
- All GC operations that are associated with this cycle occur during the STW pause.
- The partial GC cycle consists of only one increment, which runs a copy-forward operation on 512 *eden* regions and 80 other regions, followed by a class-unload operation.
- The partial GC cycle recreates a free eden space by reclaiming all possible regions from the eden space (some objects will always survive) and assigning other free regions to the eden space. The GC cycle also reclaims memory from some other regions.
- 2864 MB of the total 4096 MB heap was reclaimed. 100% of the eden space is available as free memory, and some older regions were also reclaimed.
- Between the start and end of the partial GC cycle, the remembered set count increases by 172,384 cards and the number of free bytes decreases by 0.66 MB (689,536 B). After performing a copy-forward operation on objects to move them to older regions, the partial GC cycle rebuilds the remembered set of any regions that received these moved objects. During a partial cycle, the remembered set is also pruned. Overall, the rebuilding and pruning can lead to either an increase or a decrease in the remembered set count and free memory available. 
- The remembered set metastructure remains unchanged between GC cycles, even though the mutator threads build new data about object references when the threads run. The remembered set count is identical at the end of one partial GC cycle and the beginning of the next because the remembered set consumes this data and reports to the verbose GC logs only during a cycle's operation.

### `balanced` global mark GC cycle
 
The `balanced` policy’s global *mark* GC cycle uses a mixture of STW and concurrent operations to build a new record of object liveness across the heap for use by the partial GC cycle. When an allocation taxation threshold is reached, the global mark cycle is triggered. The global *mark* cycle performs a [global *mark* phase](vgclog_balancedexamples.md#mark-phase) and also triggers an associated [*sweep* phase](vgclog_balancedexamples.md#sweep-phase) within the partial GC cycle that immediately follows the end of the global *mark* cycle.

To search for a `balanced` global mark cycle, you can search for the `type` attribute  value `global mark phase` in `<cycle-start>` and `<cycle-end>` elements.

The global cycle is split into multiple increments, each recorded as `type="global mark phase"`. A global mark phase increment involves an STW subincrement, which runs a global mark operation during an STW pause, followed by a *global mark phase(GMP) work packet* subincrement. The`GMP work packet` subincrement involves a processing operation that runs concurrently. The `GMP work packet` subincrement might also use an STW pause to complete if the subincrement is interrupted by a partial or global cycle trigger.

Splitting the global mark phase into these increments and subincrements reduces pause times by running the majority of the GC work concurrently and interleaving global mark phase increments with partial GC cycles, and, rarely, [global GC cycles](vgclog_examples.md#balanced-global-gc-cycle).

A global *mark* GC cycle increment is triggered by an [allocation taxation threshold](vgclog_examples.md#balanced-examples).

The following elements log the GC increments, subincrements and operations of the global *mark* GC cycle:

|GC increment         | GC operations| *stop-the-world* or concurrent| XML element of GC increment| Details                         |
|---------------------|-------------|-------------------------------|--------------------------------------|-----------------------|
|`global mark phase` subincrement| mark | *stop-the-world* | `<gc-start>`, `<gc-end>` |The global mark phase operations start at the beginning of the cycle and run through all *regions* until the final *region* |
|`GMP work packet processing` subincrement| work packet processing (WPP) operations | concurrent and sometimes final operations during a *STW* to complete the subincrement | `<concurrent-start>`, `<concurrent-end>`| The `GMP work packet processing subincrement` runs immediately after the `global mark phase` subincrement |
|final global mark phase increment | final global mark phase operations including class unload | *stop-the-world* | `gc-start>`, `<gc-end>`| Final increment. Runs the final global mark phase operations followed by operations to finish the cycle  |

The `balanced` global *mark* GC cycle follows a general structure in the verbose GC log as shown. The lines are indented to help illustrate the flow and some child elements are omitted for clarity:

```xml
<exclusive-start/>                        (STW pause starts)

<allocation-taxation/>                    (memory threshold trigger recorded)

<cycle-start type="global mark phase"/>   (global mark cycle starts)

<gc-start type="global mark phase"/>      (1st GMP STW subincrement starts)

    <mem-info>                            (memory status before operations)

    <remembered-set>

    </mem-info>         

</gc-start> 

<gc-op type="mark increment" />           (STW copy forward operation completed)

<gc-op  type="class unload" />            (optional STW class unload operation completed)

<gc-end>                                  (partial cycle increment ends)

    <mem-info>                            (memory status after operations)
          
    <remembered-set>

    </mem-info>

<gc-end>

<concurrent-start type="GMP work packet processing"/> (1st GMP concurrent subincrement starts)

<exclusive-end/>                                      (STW pause ends)

<concurrent-end type="GMP work packet processing"/>   (1st GMP concurrent subincrement ends)

<gc-op type="mark increment"/>                        (copy forward operation runs concurrently)

</concurrent-end type="GMP work packet processing"/>>

...                                       (partial GC cycles run during STW pauses)

<exclusive-start/>                        (STW pause starts)

<gc-start type="global mark phase"/>                  (2nd STW GMP subincrement starts)

...   

<concurrent-start type="GMP work packet processing"/> (2nd concurrent GMP subincrement starts)

...

<exclusive-end/>

...                                       (more partial GC cycles may run)

<concurrent-end type="GMP work packet processing" />  (2nd concurrent GMP subincrement ends)
...

</concurrent-end>

...                                       (more partial cycles and GMP increments interleave)

<exclusive-start/>                        (STW pause starts)

...

<gc-start type="global mark phase"/>      (final STW GMP subincrement starts.)

...   
<gc-end type="global mark phase"/>        (final STW GMP subincrement ends. No concurrent subincrement runs)

<cycle-end type="global mark phase"/>     (end of global mark cycle)

<exclusive-end/>                          (STW pause ends)    

<exclusive-start/>                        (STW pause starts)

<cycle-start type="partial gc" />         (partial cycle starts)

...

<gc-op type="sweep" />                    (Sweep operation associated with global mark cycle runs)

...

<cycle-end type="partial gc"/>            (partial GC cycle ends)

<exclusive-end/>                          (STw pause ends)

```


#### Global mark phase

The first activity of the global mark cycle is a STW pause, recorded by an `<exclusive-start>` element. The GC pauses application threads to execute the cycle's initial operations.

```xml
<exclusive-start id="1152" timestamp="2021-02-26T11:17:25.033" intervalms="1931.263">
  <response-info timems="3.082" idlems="1.041" threads="2" lastid="00000000006EDE00" lastname="RunDataWriter.1" />
</exclusive-start>
```

The cycle is triggered when the allocation taxation threshold is reached. The <allocation-taxation> element records the memory threshold value that was in use when this example global mark cycle was triggered. The`taxation-threshold` attribute shows the taxation threshold is 1024MB (1,073,741,824), which is half the total memory of the *eden* space (2048 MB), as expected for threshold triggers of global mark cycles and increments. For more information taxation thresholds for the `balanced` policy, see [`balanced` examples](vgclog_examples.md#balanced-examples).

```xml
<allocation-taxation id="1153" taxation-threshold="1073741824" timestamp="2021-02-26T11:17:25.034" intervalms="1931.251" />
```

Details about the start of the global mark GC cycle are recorded by the `<cycle-start>` element. The cycle is recorded as type `global mark phase` with an `id=1154`. Any subsequent elements associated with this cycle have a `contextid=1154` to match the global mark GC cycle's `id`. You can use the `contextid` value to distinguish increments and operations of the global mark GC cycle from the partial cycles that interleave with it.

```xml
<cycle-start id="1154" type="global mark phase" contextid="0" timestamp="2021-02-26T11:17:25.034" intervalms="374365.075" />
```

The cycle begins with the STW subincrement of a global *mark* phase increment. The STW subincrement is recorded using the `<gc-start>` element of type `global mark phase`.

```xml
<gc-start id="1155" type="global mark phase" contextid="1154" timestamp="2021-02-26T11:17:25.034">
  <mem-info id="1156" free="1442840576" total="4294967296" percent="33">
    <remembered-set count="2197888" freebytes="162912768" totalbytes="171704320" percent="94" regionsoverflowed="3" regionsstable="130" regionsrebuilding="0"/>
  </mem-info>
</gc-start>

```

The `<gc-start>` element provides a snapshot of the free memory available in the heap and the status of marked objects. At the start of the increment, the heap is 33% free, with 1376 MB (1442840576 B) of the total 4096 MB (4294967296 B) heap free.

The `<remembered-set>` element records the status of the remembered set metastructure of the JVM, a structure that records object references that cross different regions. During the rebuilding of the *remembered set* metastructure, any regions that cannot be rebuilt into a remembered set due to a lack of memory resource in the metastructure are marked as *overflow* regions. Cycles cannot reclaim memory from overflow regions.

The aim of the global mark cycle to create a new record of object liveness can be seen in the remembered set statistics. The global mark cycle populates the remembered set and also attempts to completely rebuild the set for the overflowed regions. 

The logs show that at the start of this STW subincrement, the remembered set count is 2,197,888 cards, the metastructure is using 94% of its total available memory, and three overflow regions need to be rebuilt.

The `<gc-op>` element records that the STW subincrement runs a [mark operation](gc_overview.md#gc-mark-operation). This operation begins the process of building a record of object liveness across the heap.

```xml
<gc-op id="1157" type="mark increment" timems="122.825" contextid="1154" timestamp="2021-02-26T11:17:25.157">
  <trace-info objectcount="7726701" scancount="7584109" scanbytes="213445656" />
</gc-op>
```

The `<trace-info>` element records information about the marking and scanning stages of the mark increment operation. `objectcount` records the number of objects that have been marked, ready for tracing. After marking live objects, a scan is run to trace objects and references. `scancount` records the number of marked objects that have been scanned. `scanbytes` records the total memory of all marked objects that have been scanned.

The mark increment operation in the example marked 7,726,701 objects and scanned 7,584,109 of these marked objects. The 7,584,109 of scanned objects take up 203.5 MB (213445656 B) of memory. The number of scanned objects is less than the number of marked objects because only objects that have children require scanning. Also, the scanning part of the marking operation might be interrupted by the garbage collector if a trigger threshold for a partial cycle or global cycle is reached during the marking operation.

The STW `global mark phase` subincrement ends, as recorded by `<gc-end>`, which records a snapshot of the memory status in the heap in a similar way to `<gc-start>`. 

```xml
<gc-end id="1158" type="global mark phase" contextid="1154" durationms="123.139" usertimems="977.851" systemtimems="0.000" stalltimems="1.453" timestamp="2021-02-26T11:17:25.157" activeThreads="8">
  <mem-info id="1159" free="1442840576" total="4294967296" percent="33">
    <remembered-set count="3263968" freebytes="158648448" totalbytes="171704320" percent="92" regionsoverflowed="0" regionsstable="0" regionsrebuilding="133"/>
  </mem-info>
</gc-end>
```

Comparing the snapshot at the beginning and end of this STW `global mark phase` subincrement shows that:

- The marking operation has increased the `count` value of the `<remembered-set>` by 1,066,080 cards (from 2,197,888 to 3,263,968)
- The number of overflow regions has reduced from three to zero. This GMP increment has rebuilt all overflow regions.
- As expected of a global mark cycle's operations, there is no change in the amount of free memory available, which is 1376 MB.

The beginning of the second part of the global *mark* phase increment, the GMP work packet procesing subincrememt, is recorded by `<concurrent-start>`. The child element `<concurrent-mark-start>` records the scan target of this subincrement as 242.74 MB (254,532,672 B).

```xml
<concurrent-start id="1160" type="GMP work packet processing" contextid="1154" timestamp="2021-02-26T11:17:25.157">
  <concurrent-mark-start scanTarget="254532672" />
</concurrent-start>
```

Now that the STW global mark phase sub-increment is complete, application threads are restarted. 

```xml
<exclusive-end id="1161" timestamp="2021-02-26T11:17:25.157" durationms="123.936" />
```

The `GMP work packet processing` sub-increment continues to run concurrently. The end of this subincrement's operations are recorded using the `<concurrent-end>` element. 

```xml
<concurrent-end id="1162" type="GMP work packet processing" contextid="1154" timestamp="2021-02-26T11:17:25.469" terminationReason="Work target met">
<gc-op id="1163" type="mark increment" timems="311.867" contextid="1154" timestamp="2021-02-26T11:17:25.469">
  <trace-info scanbytes="254708852" />
</gc-op>
</concurrent-end>
```

The child element `<trace-info>` shows that the processing has scanned 242.91 MB (254,708,852 B), which exceeds the 108.25 MB scan target.

The garbage collector now returns to running partial cycles to reclaim free space in the heap before the next global mark phase increment is triggered. To see an example of how a `balanced` partial GC cycle appears in the logs, see the [`Balanced` examples - Partial GC Cycle](vgclog_examples#partial-gc-cycle).

Following some partial GC cycles, an allocation taxation threshold is reached which triggers a STW pause followed by another global mark phase increment. The element `<gc-start>` in the following log excerpt has a `contextid=1154` and type `global mark phase`, which indicates that this is a global mark phase sub-increment associated with our global *mark* cycle example.

```xml
<exclusive-start id="1175" timestamp="2021-02-26T11:17:28.993" intervalms="1978.886">
  <response-info timems="5.111" idlems="1.714" threads="2" lastid="00000000006EDE00" lastname="RunDataWriter.1" />
</exclusive-start>
<allocation-taxation id="1176" taxation-threshold="1073741824" timestamp="2021-02-26T11:17:28.994" intervalms="1978.879" />
<gc-start id="1177" type="global mark phase" contextid="1154" timestamp="2021-02-26T11:17:28.994">
  <mem-info id="1178" free="1451229184" total="4294967296" percent="33">
    <remembered-set count="3325824" freebytes="158401024" totalbytes="171704320" percent="92" regionsoverflowed="2" regionsstable="0" regionsrebuilding="133"/>
  </mem-info>
</gc-start>
```

The `<allocation-taxation>` element shows the taxation threshold for triggering this global mark phase increment is set to 1024 MB, half of the size of the *eden* space, as expected. 

`<gc-start>` records the heap to have 1384 MB (1,451,229,184 B)of free memory available at the beginning of this global mark phase increment. This compares to the 1376 MB (1,442,840,576 B) of free memory available at the end of the previous global mark phase increment. Although free memory was reclaimed by the partial GC cycles that ran between these global mark phase increments, free memory was allocated to objects when application threads ran, resulting in a net reduction of free memory available.

The `<remembered set>` element shows that there are two overflow regions to rebuild.

The status of the heap at the beginning and end of STW sub-increments are automatically recorded. For this STW subincrement, there are no `<gc-op>` elements recorded; `<gc-end>` immediately follows `<gc-start>` in the logs. For some STW sub-increments, some GC operations are run, such as ADD

```xml
<gc-end id="1179" type="global mark phase" contextid="1154" durationms="0.289" usertimems="1.000" systemtimems="0.000" stalltimems="0.000" timestamp="2021-02-26T11:17:28.994" activeThreads="8">
  <mem-info id="1180" free="1451229184" total="4294967296" percent="33">
    <remembered-set count="3325824" freebytes="158401024" totalbytes="171704320" percent="92" regionsoverflowed="2" regionsstable="0" regionsrebuilding="133"/>
  </mem-info>
</gc-end>
```

Comparing the heap status at the beginning and end of the subincrement shows that there is no change in free memory available or `<remembered-set>`  values.

The second part of the increment, the `GMP work packet processing` subincrement, is recorded using the `<concurrent-start>` and <concurrent-end>` elements. 

```xml
<concurrent-start id="1181" type="GMP work packet processing" contextid="1154" timestamp="2021-02-26T11:17:28.994">
  <concurrent-mark-start scanTarget="258671414" />
</concurrent-start>
<exclusive-end id="1182" timestamp="2021-02-26T11:17:28.994" durationms="0.816" />

<concurrent-end id="1183" type="GMP work packet processing" contextid="1154" timestamp="2021-02-26T11:17:29.273" terminationReason="Work target met">
<gc-op id="1184" type="mark increment" timems="279.311" contextid="1154" timestamp="2021-02-26T11:17:29.274">
  <trace-info scanbytes="258767612" />
</gc-op>
</concurrent-end>
```

The log excerpt shows the concurrent `GMP work packet processing` subincrement has achieved the scan target of 246.69 MB (258671414 B). 246.78 MB (258767612 B) were scanned.

More partial cycles run. This pattern of interleaving of global mark increments with partial gc cycles repeats until a final global mark increment completes the global mark cycle. The final global mark phase increment consists of a STW `global mark phase` sub-increment that includes `mark increment` and `class unload` operations.

```xml
<exclusive-start id="1217" timestamp="2021-02-26T11:17:36.864" intervalms="1986.124">
  <response-info timems="0.287" idlems="0.104" threads="2" lastid="00000000006EDE00" lastname="RunDataWriter.1" />
</exclusive-start>
<allocation-taxation id="1218" taxation-threshold="1073741824" timestamp="2021-02-26T11:17:36.865" intervalms="1986.101" />
<gc-start id="1219" type="global mark phase" contextid="1154" timestamp="2021-02-26T11:17:36.865">
  <mem-info id="1220" free="1438646272" total="4294967296" percent="33">
    <remembered-set count="3514496" freebytes="157646336" totalbytes="171704320" percent="91" regionsoverflowed="3" regionsstable="0" regionsrebuilding="133"/>
  </mem-info>
</gc-start>
<gc-op id="1221" type="mark increment" timems="164.843" contextid="1154" timestamp="2021-02-26T11:17:37.030">
  <trace-info objectcount="7715572" scancount="7665293" scanbytes="214739196" />
  <cardclean-info objects="3962203" bytes="117924792" />
  <finalization candidates="206" enqueued="30" />
  <ownableSynchronizers candidates="601780" cleared="16925" />
  <references type="soft" candidates="718240" cleared="2858" enqueued="2832" dynamicThreshold="18" maxThreshold="32" />
  <references type="weak" candidates="2321" cleared="142" enqueued="0" />
  <references type="phantom" candidates="8" cleared="0" enqueued="0" />
  <stringconstants candidates="9522" cleared="0"  />
  <object-monitors candidates="7142" cleared="7066"  />
</gc-op>
<gc-op id="1222" type="classunload" timems="0.704" contextid="1154" timestamp="2021-02-26T11:17:37.030">
  <classunload-info classloadercandidates="185" classloadersunloaded="13" classesunloaded="13" anonymousclassesunloaded="0" quiescems="0.000" setupms="0.644" scanms="0.043" postms="0.016" />
</gc-op>
<gc-end id="1223" type="global mark phase" contextid="1154" durationms="169.521" usertimems="1244.810" systemtimems="3.000" stalltimems="27.792" timestamp="2021-02-26T11:17:37.034" activeThreads="8">
  <mem-info id="1224" free="1438646272" total="4294967296" percent="33">
    <pending-finalizers system="30" default="0" reference="2832" classloader="0" />
    <remembered-set count="2241440" freebytes="162738560" totalbytes="171704320" percent="94" regionsoverflowed="3" regionsstable="127" regionsrebuilding="0"/>
  </mem-info>
</gc-end>
```

Comparing the status of the memory at the start and end of this final `global mark phase` increment shows that:

- As expected, the final global mark phase increment does not reclaim any free memory.
- The remembered set metastructure has been rebuilt. The count has reduced and the amount of available memory in the metastructure has increased from 91% to 94%.
- The number of overflow regions remains unchanged. The final global mark phase increment did not manage to rebuild any overflow regions.

Following the final global mark increment, the global mark cycles completes and the GC ends the STW pause.

```xml
<cycle-end id="1225" type="global mark phase" contextid="1154" timestamp="2021-02-26T11:17:37.034" />
<exclusive-end id="1226" timestamp="2021-02-26T11:17:37.034" durationms="170.186" />

```

The operations to create a record of object liveness across the heap, which began with the global *mark* cycle, is completed with a sweep phase. The sweep phase is triggered by the end of the global *mark* cycle to be included in the next partial GC cycle that runs.

#### Sweep phase

While the global *sweep* operation is logically associated with the global *mark* phase, it does not run in the same global *mark* cycle. Instead, the *sweep* operation runs in the same STW increment as the first partial GC cycle that runs after the completion of the global *mark* cycle. This can be seen in the following log excerpt.  After the log records the end of the global mark cycle it records a STW pause followed by a `partial gc` cycle of `id=1229`. The global *sweep* operation that runs after the the global *mark* phase is recorded in the `<gc-op>` element tagged as `id=1229`.

```xml
<exclusive-start id="1227" timestamp="2021-02-26T11:17:38.804" intervalms="1940.125">
...
<cycle-start id="1229" type="partial gc" contextid="0" timestamp="2021-02-26T11:17:38.805" intervalms="3926.202" />
...
</gc-start>
...
</gc-start>
<gc-op id="1232" type="sweep" timems="9.472" contextid="1229" timestamp="2021-02-26T11:17:38.815" />
<gc-op id="1233" type="copy forward" timems="308.258" contextid="1229" timestamp="2021-02-26T11:17:39.124">
...
<gc-op id="1234" type="classunload" timems="0.012" contextid="1229" timestamp="2021-02-26T11:17:39.125">
...
<gc-end>
...
</gc-end>
<cycle-end id="1237" type="partial gc" contextid="1229" timestamp="2021-02-26T11:17:39.125" />
<exclusive-end id="1238" timestamp="2021-02-26T11:17:39.125" durationms="320.792" />
```

A record of object liveness is now complete.
*
#### Summary of the example

Analyzing the structure and elements of this example log output shows that this example `balanced` global mark GC cycle has the following characteristics:

- The GC cycle begins with an STW pause and is triggered by a taxation allocation threshold that has a value equal to half the size of the eden space. Each global mark phase is also triggered by a taxation threshold equal to the size of half of the eden space.
- The global *mark* cycle does not reclaim memory. The cycle creates a record of object liveness, which can be seen by inspecting the status of the remembered set metastructure using the `<remembered-set>` attributes.
- Each global mark phase increment rebuilds the remembered set metastructure. Sometimes, the increment has managed to rebuilt some of the overflow regions.
- Partial cycles run in between global mark phase increments.
- The final global mark phase increment includes a class unload. The final increment also triggers a sweep phase to run in the next partial cycle.

### WIP!!! `balanced` global cycle WIP!!!

The following [`balanced` partial GC cycle](gc.md#balanced-policy) example is taken from a `balanced` verbose GC log. The output is broken down into sections to explain the GC processing that is taking place.

To search for a `balanced` global cycle, you can search for the `type` attribute value `global garbage collect` in `<cycle-start>` and `cycle-end` elements. You can also search for the `sys-start` element, which records the `balanced` global cycle trigger as `explicit`.

If the `balanced` policy GC cannot reclaim enough memory using the `balanced` partial and global *mark* cycles to prevent the heap becoming completely full, an allocation failure occurs and a `balanced` global cycle is triggered.

The ['balanced` global cycle](gc.md)'s operations run during in a single GC increment during a STW pause.

CHANGE TABLE TO HTML, ACCESSIBLE TABLE
|GC Operations | GC increment | *stop-the-world* or concurrent| XML element of GC increment          | Details                                                                   |
|-------------|--------------|-------------------------------|--------------------------------------|---------------------------------------------------------------------------|
|???? |Single        | *stop-the-world*              | `<gc-start>`, `<gc-end>`| `<gc-op'>` |ADD| 

The `balanced` partial GC cycle follows a general structure in the verbose GC log as shown. The lines are indented to help illustrate the flow and some child elements are omitted for clarity:

CREATE THIS ILLUSTRATION FOR GLOBAL CYCLE - THIS IS CURRENTLY COPIED FROM PARTIAL GC

```xml
<exclusive-start/>                       (STW pause starts)

<allocation-taxation/>                   (memory threshold trigger recorded)

<cycle-start/>                            (partial cycle starts)

<gc-start/>                             (partial  cycle increment starts)

<mem-info>                            (memory status before operations)

 <mem></mem>                         (status of different types of memory)

</mem-info>         

</gc-start> 

<allocation-stats>                      (snapshot of application threads status...
                                          ... taken before the cycle starts)
<gc-op> type="copy forward" </gc-op>    (copy forward operation completed)

<gc-op> type="class unload" </gc-op>    (class unload operation completed)

<gc-op> type="sweep" </gc-op>           (sweep operation completed)

<gc-op> type="compact" </gc-op>         (compact operation completed)

<gc-end>                                (partial cycle increment ends)

<mem-info>                            (memory status after operations)
          
<mem></mem>                         (status of different types of memory)

</mem-info>         

</gc-end>

<cycle-end>                               (partial cycle ends)

<exclusive-end>                           (STW pause ends)
```

REPLACE WITH A GLOBAL CYCLE THAT IS MUCH LATER ON IN THE LOG AND WITH ONE THAT ALEKSANDAR FINDS ME - AN IMPLICIT TRIGGERED ONE!
The first activity of a global cycle is a STW pause, recorded by an `<exclusive-start` element. The following `<sys-start>` element records the trigger reason as `explicit`.

```xml
<exclusive-start id="41" timestamp="2020-11-13T06:31:42.006" intervalms="2429.574">
  <response-info timems="2.338" idlems="0.802" threads="2" lastid="000000000074FF00" lastname="RunDataWriter.1" />
</exclusive-start>
<sys-start reason="explicit" id="42" timestamp="2020-11-13T06:31:42.006" intervalms="9987.458" />
```

The start of the cycle is recorded using the `<cycle-start>` element as type `global garbage collect`.

```xml
<cycle-start id="43" type="global garbage collect" contextid="0" timestamp="2020-11-13T06:31:42.007" intervalms="9987.772" />
```

The 

```xml
<gc-start id="44" type="global garbage collect" contextid="43" timestamp="2020-11-13T06:31:42.007">
  <mem-info id="45" free="2281852504" total="3221225472" percent="70">
    <mem type="eden" free="151128" total="130023424" percent="0" />
    <arraylet-reference objects="4" leaves="4" largest="1" />
    <arraylet-primitive objects="1" leaves="8" largest="8" />
    <remembered-set count="453472" freebytes="126964352" totalbytes="128778240" percent="98" regionsoverflowed="4" regionsstable="0" regionsrebuilding="0"/>
  </mem-info>
</gc-start>
```

```xml
<allocation-stats totalBytes="127765576" >
  <allocated-bytes non-tlh="11566360" tlh="116199216" arrayletleaf="0"/>
  <largest-consumer threadName="ForkJoinPool-3-worker-5" threadId="000000000077E900" bytes="53174768" />
</allocation-stats>
```

```xml
<gc-op id="46" type="global mark" timems="35.727" contextid="43" timestamp="2020-11-13T06:31:42.044">
  <trace-info objectcount="1856388" scancount="1838558" scanbytes="66324640" />
  <finalization candidates="87" enqueued="0" />
  <ownableSynchronizers candidates="200093" cleared="200034" />
  <references type="soft" candidates="4427" cleared="0" enqueued="0" dynamicThreshold="23" maxThreshold="32" />
  <references type="weak" candidates="2968" cleared="31" enqueued="29" />
  <references type="phantom" candidates="97" cleared="0" enqueued="0" />
  <stringconstants candidates="8948" cleared="0"  />
  <object-monitors candidates="20" cleared="7"  />
</gc-op>
<gc-op id="47" type="classunload" timems="0.020" contextid="43" timestamp="2020-11-13T06:31:42.044">
  <classunload-info classloadercandidates="74" classloadersunloaded="0" classesunloaded="0" anonymousclassesunloaded="0" quiescems="0.000" setupms="0.020" scanms="0.000" postms="0.000" />
</gc-op>
<gc-op id="48" type="sweep" timems="2.322" contextid="43" timestamp="2020-11-13T06:31:42.047" />
<gc-op id="49" type="compact" timems="66.804" contextid="43" timestamp="2020-11-13T06:31:42.114">
  <compact-info movecount="1651540" movebytes="56069168" />
  <remembered-set-cleared processed="37569" cleared="33373" durationms="1.000" />
</gc-op>
```

```xml
<gc-end id="50" type="global garbage collect" contextid="43" durationms="107.552" usertimems="688.000" systemtimems="36.000" stalltimems="130.908" timestamp="2020-11-13T06:31:42.115" activeThreads="8">
  <mem-info id="51" free="3128950784" total="3221225472" percent="97">
    <arraylet-reference objects="2" leaves="2" largest="1" />
    <arraylet-primitive objects="1" leaves="8" largest="8" />
    <pending-finalizers system="0" default="0" reference="29" classloader="0" />
    <remembered-set count="26656" freebytes="128671616" totalbytes="128778240" percent="99" regionsoverflowed="0" regionsstable="0" regionsrebuilding="0"/>
  </mem-info>
</gc-end>
```

```xml
<cycle-end id="52" type="global garbage collect" contextid="43" timestamp="2020-11-13T06:31:42.115" />
<sys-end id="53" timestamp="2020-11-13T06:31:42.115" />
<exclusive-end id="54" timestamp="2020-11-13T06:31:42.115" durationms="108.749" />
```

#### Summary of the example
ADD HERE

- The cycle has reclaimed memory in the heap that the pgc cannot reclaim
- Rebuilt overflow regions
-