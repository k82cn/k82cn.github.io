---
layout: post
categories: tech
title: ACE(2) ACE_Process
---

ACE provides lots of wrapper classes to hide the difference between OS. ACE_Process is one of them to give us an unified operation interface to fork/kill/wait a process. And ACE_Process is also a good example code to show useful function, such as how to check whether a process is alive.

## ACE_Process_Options

The ACE_Process_Options is a class that used to set child process’s attributes, such as command line, arguments and environment. In OGL, command_line is used to set task’s binary:

int ACE_Process_Options::command_line(const ACE_ANTI_TCHAR* format, ... );

In JobRunner, m_taskProcessOption points to an ACE_Process_Options object; this member is updated when JobRunner service to a Job (bindJob):

    releaseObject<ACE_Process_Options>(m_taskProcessOption);
    ACE_NEW_RETURN(m_taskProcessOption, ACE_Process_Options(), -1);
    m_taskProcessOption->command_line(jobOption->command());

## Spawn child process

ACE_Process is a set of process’s runtime attribute, such as pid; and it’s also a collection of child process operations. JobRunner uses ACE_Process::spawn(…) to execute task:

    task.spawn(*(this->m_taskProcessOption));

## Re-direct stdio

Pipe is a easy way for parent/child process to exchange data; in OGL, the input/output handles of child process are re-directed to a pipe, which is used to get task input from JobRunner and send the task output back.

    ACE_HANDLE taskOutput[2];
    ACE_HANDLE taskInput[2];

    ACE_OS::pipe(taskOutput);
    ACE_OS::pipe(taskInput);

    this->m_taskProcessOption->release_handles();

    this->m_taskProcessOption->set_handles(taskInput[OGL_PIPE_READ],
                                           taskOutput[OGL_PIPE_WRITE]);

    ACE_OS::close(taskInput[OGL_PIPE_READ]);
    ACE_OS::close(taskOutput[OGL_PIPE_WRITE]);
    
    task.close_dup_handles();
    task.close_passed_handles();
    
    // release the duplicated handles
    m_taskProcessOption->release_handles();

## Child process operation

Wait child process: just like the wait function in Linux, there is also a wait function in ACE_Process for parent process to synchronize with its child process.

    task.wait();

Is child process running: ACE_Process also provide a function to check whether the child process is alive, named running(). The implementing of running function in Linux is kill(pid, 0).

    // Return 1 if running; 0 otherwise.
    int running (void) const

If sig is 0, then no signal is sent, but error checking is still performed; this can be used to check for the existence of a process ID or process group ID.

Reference [man kill](http://linux.die.net/man/2/kill) for more details.

Code in OGL

    int JobRunner::bindJobRunner(ogl::CommandHeader& header, ogl::JobOption* jobOption)
    {
        OGL_LOG_DEBUG("bind job runner for job id: <%d>, runner id: <%s>",
                      (int)jobOption->id(),
                      jobOption->runnerId());
    
        releaseObject<ACE_Process_Options>(m_taskProcessOption);
    
        ACE_NEW_RETURN(m_taskProcessOption, ACE_Process_Options(), -1);
    
        m_taskProcessOption->command_line(jobOption->command());
    
        CommandHeader respHeader(BindJobRunnerComplete, jobOption->runnerId());
    
        return sendResponse(respHeader, jobOption);
    }
    
    int JobRunner::executeTask(ogl::CommandHeader& header, ogl::TaskOption* taskOption)
    {
    
        OGL_LOG_DEBUG("Execute task for job id: <%d>, task id: <%d>, runner id: <%s>",
                      (int)taskOption->jobId(),
                      (int)taskOption->taskId(),
                      taskOption->runnerId());
    
        ACE_Process task;
    
        ACE_HANDLE taskOutput[2];
        ACE_HANDLE taskInput[2];
    
        ACE_OS::pipe(taskOutput);
        ACE_OS::pipe(taskInput);
    
        this->m_taskProcessOption->release_handles();
    
        this->m_taskProcessOption->set_handles(taskInput[OGL_PIPE_READ],
                                               taskOutput[OGL_PIPE_WRITE]);
    
        ogl::File inputStream(taskInput[OGL_PIPE_WRITE]);
        ogl::File outputStream(taskOutput[OGL_PIPE_READ]);
    
        task.spawn(*(this->m_taskProcessOption));
    
        ACE_OS::close(taskInput[OGL_PIPE_READ]);
        ACE_OS::close(taskOutput[OGL_PIPE_WRITE]);
    
        task.close_dup_handles();
        task.close_passed_handles();
    
        // release the duplicated handles
        m_taskProcessOption->release_handles();
   
        ogl::Buffer& input = taskOption->taskInput();
        if (input.size() > 0) {
            inputStream.write(input);
        }
    
        ogl::Buffer& output = taskOption->taskOutput();
        outputStream.read(output);
   
        task.wait();
   
        CommandHeader respHeader(ExecuteTaskComplete, header.contextId());
    
        return sendResponse(respHeader, taskOption);
    }
