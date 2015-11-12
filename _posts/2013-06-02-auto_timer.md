---
layout: post
categories: tech
title: Auto Timer in OGL
---

For the performance tuning, the simplest way is to record how many time is elapsed in a function. The only difficulty we’re facing is that: there maybe many exit for a function. Thanks to C++’s constructor/deconstructor feature, it’s easy for developer to record the elsaped time.

## auto timer in C++

Compared to C, constructor/deconstructor is an important improvement of C++. It’s not only make sure that the object/struct will be automatically initialized, but also keep C++ developer far away from MLK issue by Smart Pointer. And it’s also a good guidance for developer to do simple performance tuning; here is a implemention of the auto timer:

    class AutoTimer
    {
        public:
            AutoTimer(const char* label) : m_label(label)
            {
                m_start = new timeval();
                gettimeofday((timeval*)m_start, NULL);
            }

            ~AutoTimer()
            {
                m_end = new timeval();
                gettimeofday((timeval*)m_end, NULL);

                double start, end;
    
                start = ((timeval*)m_start)->tv_sec * 1000 + ((timeval*)m_start)->tv_usec / 1000.0;
                end = ((timeval*)m_end)->tv_sec * 1000 + ((timeval*)m_end)->tv_usec / 1000.0;
    
                if (logger) {
                    loggerDrv->log2(__FILE__, __LINE__, "%s : start: <%lf>, end: <%lf>, duration: <%lf>.", 
                            m_label.c_str(), start, end, end - start);
                }

                delete (timeval*) m_start;
                delete (timeval*) m_end;
            }

        private:
            timeval* m_start;
            timeval* m_end;
            std::string m_label;
    };

## How to use

Now, we have a simple performance testing tool: auto timeer; and the it’s easy to use as follow:

    int handle_input(ogl::Handler fd)
    {
        AutoTimer timer("handle_input");
        ....
    }
