---
layout: post
title: protobuf&#58; default value of optional enum
categories: tech
---

## Summary

The default value is helpful when using `enum` in protobuf; but there're some tips I'd like to highlight here:

1. If the modifier is `required`, `set_xx()` is still necessary before serializing
2. `xx()` will return the default value
3. `has_xx()` still return false if `set_xx()` is not executed

## Example of protobuf

    message Msg {
        enum Type {
            A = 1;
            B = 2;
        }

        optional Type type = 1 [ default = B ];
    }

The following code slice shows that the default value of type will be `B`, and `has_type()` returns false if `set_type()` is not executed.

    #include <iostream>
    #include "opt.pb.h"
    
    using namespace std;
    
    int main(int argc, char** argv)
    {
        Msg msg;
        if (msg.has_type()) {
            cout << "default type is Enabled!" << endl;
        } else {
            cout << "default type is Optional" << endl;
        }

        string buf;

        msg.SerializeToString(&buf);

        msg.ParseFromString(buf);

        if (msg.has_type()) {
            cout << "default type is Enabled!" << endl;
        } else {
            cout << "default type is Optional" << endl;
        }

        Msg::Type t = msg.type();

        cout << t << endl;

        msg.set_type(Msg::A);
        if (msg.has_type()) {
            cout << "default type is Enabled!" << endl;
        } else {
            cout << "default type is Optional" << endl;
        }

        return 0;
    }

