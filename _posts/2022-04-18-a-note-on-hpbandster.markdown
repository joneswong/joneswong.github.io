---
layout: post
title:  "A Note on HPO package hpbandster"
date:   2022-04-18 20:24:02 -0700
categories: Python HPO
---

Let's understand this package by going through the provided ![example](https://automl.github.io/HpBandSter/build/html/auto_examples/example_1_local_sequential.html).
For simplicity, we begin with ![random search optimizer](https://automl.github.io/HpBandSter/build/html/optimizers/randomsearch.html) rather than the BOHB employed in this example.

## Preliminary

We must understand python's `threading.Condition` before diving into the implementation of hpbandster.

At first, `threading.Condition` has methods `acquire()` and `release()` and obeys the context management protocol:

  > All of the objects provided by this module (i.e., threading) that have acquire() and release() methods can be used as context managers for a with statement. The acquire() method will be called when the block is entered, and release() will be called when the block is exited.

Please see ~[python document](https://docs.python.org/3/library/threading.html) for more details.


## The architecture of optimizers

At first, the base class for all these optimizers is `Master` class (see hpbandster/core/master.py), which utilizes a `Dispatcher` object for

  > assigning tasks to free workers, report results back to the master and communicate to the nameserver.

Let's see how it achieves this from the construction:
```python
self.dispatcher = Dispatcher( self.job_callback, queue_callback=self.adjust_queue_size, run_id=run_id, ping_interval=ping_interval, nameserver=nameserver, nameserver_port=nameserver_port, host=host)
```

The first argument is `Master`'s method `job_callback()`, which takes in a `Job` object (see hpbandster/core/dispatcher.py) once that job is finished, and do some "book keeping", e.g., `self.num_running_jobs -= 1`, `self.iterations[job.id[0]].register_result(job)`, `self.config_generator.new_result(job)`, and `self.thread_cond.notify()`.

The argument `queue_callback` is specified as `Master`'s method `adjust_queue_size()`, which

  > gets called with the number of workers in the pool on every update-cycle

It accordingly updates `Master`'s `job_queue_sizes` attribute and then notify all the threads waiting for the condition (i.e., `self.thread_cond.notify_all()`).

The `run()` method of `Dispatcher` object is immediately called after its instantiation, which triggers two threads: one runs `discover_workers()` and the other runs `job_runner()`.


## Going through the optimization procedure

The `run()` method of `Master` is the entry of the whole optimization procedure:
```python
def run(self, n_iterations=1, min_n_workers=1, iteration_kwargs = {},):
                """
                        run n_iterations of SuccessiveHalving

                Parameters
                ----------
                n_iterations: int
                        number of iterations to be performed in this run
                min_n_workers: int
                        minimum number of workers before starting the run
                """

                self.wait_for_workers(min_n_workers)

                iteration_kwargs.update({'result_logger': self.result_logger})

                if self.time_ref is None:
                        self.time_ref = time.time()
                        self.config['time_ref'] = self.time_ref

                        self.logger.info('HBMASTER: starting run at %s'%(str(self.time_ref)))

                self.thread_cond.acquire()
                while True:

                        self._queue_wait()

                        next_run = None
                        # find a new run to schedule
                        for i in self.active_iterations():
                                next_run = self.iterations[i].get_next_run()
                                if not next_run is None: break

                        if not next_run is None:
                                self.logger.debug('HBMASTER: schedule new run for iteration %i'%i)
                                self._submit_job(*next_run)
                                continue
                        else:
                                if n_iterations > 0:    #we might be able to start the next iteration
                                        self.iterations.append(self.get_next_iteration(len(self.iterations), iteration_kwargs))
                                        n_iterations -= 1
                                        continue

                        # at this point there is no imediate run that can be scheduled,
                        # so wait for some job to finish if there are active iterations
                        if self.active_iterations():
                                self.thread_cond.wait()
                        else:
                                break

                self.thread_cond.release()

                for i in self.warmstart_iteration:
                        i.fix_timestamps(self.time_ref)

                ws_data = [i.data for i in self.warmstart_iteration]

                return Result([copy.deepcopy(i.data) for i in self.iterations] + ws_data, self.config)
```

`wait_for_workers()` blocks the execution until there is enough free workers, where the `self.thread_cond.wait(1)` will be notified by the `self.thread_cond.notify()` in `job_callback()`.
`self.result_logger` is used to make live logging (more details can be found ![here](https://automl.github.io/HpBandSter/build/html/advanced_examples.html#live-logging)).
`self.time_ref` is set to be `None` in the constructor of `Master` and thus becomes the current moment here.
`self.thread_cond` is an object of python's `threading.Condition`, which is used for coordinating the threads.
At the first time we enter the `while` loop, the `_queue_wait()` method will not block the execution, as `job_queue_sizes` has been changed from `(-1, 0)` to `(0, 1)` by `adjust_queue_size()` called by `discover_workers()`, where there is one worker in this example.
`self.iterations` is a list intending to hold `n_iterations` iterations (each is a `SuccessiveHalving` object).
At the first time we enter the `while` loop, `active_iterations()` cannot find any active iteration. Thus, `next_run` is `None`, and the `SuccessiveHalving` object is returned by the `get_next_iteration()` method.

By `continue`, we enter the `while` loop again and come to the first `for` loop.
The base class of `SuccessiveHalving` is `BaseIteration` class (see hpbandster/core/base_iteration.py), which has attribute `is_finished` (`False` by its construction). `active_iterations()` returns `[0]` (i.e., implying that the first iteration is active), and this line `next_run = self.iterations[i].get_next_run()` is executed.
By this calling, then the `SuccessiveHalving` object returns a tuple consisting of:
- `config_id`: a tuple where the first element is the iteration index of the `Master`, the second element is the stage index of the `SuccessiveHalving` object (starting from zero), and the config index among the considered configs at this stage.
- the config
- the budget assigned to this config

Then the returned value is fed into this method `self._submit_job(*next_run)` so that the dispatcher can submit the job to the nameserver, where `num_running_jobs` is increased by one.

By `continue`, we enter the `while` loop again. The `_queue_wait()` method will block the execution untill the submitted job has finished, that is, `num_running_jobs` becomes zero by the update made by `job_callback()`.

The procedure goes on in this way.
