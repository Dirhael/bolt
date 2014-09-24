# Bolt

He is a fast runner.

Reads tasks from a mongo table called `task_queue` on a db called
`bolt_#{CURRENT_ENV}`, where `CURRENT_ENV` can be one of `production`,
`development`(the default) or `test`.

Tasks should be defined on separate files inside `bolt/tasks` folder, which
should be itself placed somewhere in load path (ex. inside `lib`).

Bolt will fork itself, and then it will load the ruby file placed in
`bolt/tasks/#{task}.rb`, where `task` is the value of the `task` field in the
mongo document. That file should define a module named with the camelized
version of the task's name. Bolt will call a method `run` that must be defined
for that module, passing the complete mongo document for this task.

Inside `bolt/tasks/my_example_task.rb` things look like this:

    module Bolt
      module Tasks
        module MyExampleTask

          def self.run(task:)
            do_something_with task
          end

        end
      end
    end

Bolt will use that same mongo document afterwards to send success (or failure)
emails depending on whether the execution ended normally or any exception was
raised.

Tasks are isolated on their own process, so they have freedom to do nasty
things. They won't corrupt Bolt or the other tasks.

Fields on the mongo document and used by Bolt are:

* `task`: the name of the task, also of the module and the file defining it.
* `timeout`: timeout for the task to end. Default is 300 secs.
* `dispatched`: boolean indicating whether Bolt started processing that task.
* `success`: when defined, indicates the result of the execution.
* `ex`, `backtrace`: failure details when the task fails.
* `email`: email recipient for the notification emails.
* `run_at`: timestamp for the start of the task. When it exists, task will not
be started by Bolt until it's in the past.


