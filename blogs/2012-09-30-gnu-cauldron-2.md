转换时间： 2021-01-31

# GNU Tools Cauldron (2)
Posted on 2012/09/30 by yao

[抱歉，没有中文的，大家凑合看看吧，这个本来是我内部用的，改了改，贴这里了]

周一， RMS gives a keynote on Free software, nothing new compared with his previous speeches I saw online, except that he tries to sell some icons and t-shirts. The organizer told me that RMS was not invited for this event, but he asked them to give a speech in Prague.

PHOTO

Robert Dewar, CEO of adacore, gives a speech on their business model, which is quite similar to codesourcery’s. The difference is on customers, adacore’s customers are military-related, such as lockheed martin, while CS’s customers are IC companies.

PHOTO

中间break的时候，让我碰到了一个让我感动的事情。Stan （中间）给Ian （左边）说，我给你带了一个cygnus （cygnus是多牛的公司，就不用我说了）的书包，本来应该16年前就给你了，今天给你带来了，一个全新的书包。 Ian很惊讶的表情，然后跑回到自己的座位，拿出自己的书包，竟然也是cygnus的！

Follow some sessions presented by Richard Guenther (on high-level loop opt), Cary (on  Fission), and Bill Schmidt (from IBM, on strength reducing). Don’t understand them fully :(.  Jeremy Bennett gives a presentation on optimization for power/energy consumption. The topic sounds pretty cool, but unfortunately, they don’t have any useful data or results to show. Some audience said something similar has been done in some papers 3-5 years ago. 🙂  这个简直就是，自己什么还都没有做，就来说，我们打算做什么，问道细节的东西，一个没有，也很失望。

There are two GDB-related sessions, “Variable Tracking in GCC” and “GDB vs. MPI”. Alexandre gives the former one. I don’t get used to his presentation style, so don’t understand the details. Two points I get are 1) there are a lot of traps in the algorithm/code, 2) he is not going to maintain it.  🙂 The MPI session is interesting, but the presenter is not good at attracting people. They extend pretty printer in GDB/python, collect data through inferior-call, and show data somewhere. Looks they don’t have a plan to contribute it upstream.  这两个session让我很失望，不知道在说什么。

During the lunch, I joined in Pedro and Ulrich’s discussion on fixing parity of GDB remote and native debugging. Both linaro and redhat want to do that.  They plan to use the same backend in both GDB and GDBserver in the long-terms, and want to get rid of libthread_db.so. However, as usual, don’t know what the patches look like in near-terms.  虽然他们说的很多东西，我都听不懂吧，但是觉得就听听他们这样的交流，也很好。
