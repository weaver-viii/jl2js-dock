
FROM ubuntu:16.04

RUN dpkg --add-architecture i386

# Install packages needed for running 32-bit Julia and using emscripten
RUN apt-get update \
 && apt-get install -y build-essential gcc g++ cmake curl gfortran git m4 python xz-utils bzip2 nodejs \
                       libc6:i386 libstdc++6:i386 zlib1g:i386 dpkg-dev:i386 \
 && ln -s nodejs /usr/bin/node

# Get the most recent emsdk
RUN curl -sSL https://s3.amazonaws.com/mozilla-games/emscripten/releases/emsdk-portable.tar.gz -o emsdk-portable.tar.gz
RUN tar -xzf  emsdk-portable.tar.gz 

# Set up emscripten -- use 1.37.6 to match LLVM versions with Julia
RUN cd /emsdk-portable \
 && ./emsdk update \
 && ./emsdk install sdk-1.37.6-64bit \
 && ./emsdk activate sdk-1.37.6-64bit \
 && /bin/bash /emsdk-portable/emsdk_env.sh \
 && find /emsdk-portable -name "llvm-nm" -exec ln -s {} /usr/bin/llvm-nm  ";" \
 && find /emsdk-portable/emscripten/ -maxdepth 2 -executable -type f -name 'em*' -exec ln -s {} /usr/bin/ \; 
RUN emcc --version \
 && mkdir -p /tmp/emscripten_test && cd /tmp/emscripten_test \
 && printf '#include <iostream>\nint main(){std::cout<<"HELLO"<<std::endl;return 0;}' > test.cpp \
 && em++ -O2 test.cpp -o test.js && node test.js \
 && em++ test.cpp -o test.js && node test.js \
 && em++ -O2 test.cpp -o test.js -s BINARYEN=1 \
 && em++ test.cpp -o test.js -s BINARYEN=1 \
 && cd / \
 && rm -rf /tmp/*

# RUN curl -sSL https://julialang-s3.julialang.org/bin/linux/x86/0.6/julia-0.6.0-linux-i686.tar.gz -o julia.tar.gz
# RUN tar -xzf  julia.tar.gz -C /usr --strip-components 1 
ADD julia32.tar.gz /
# RUN tar -xzf  julia32.tar.gz -C /usr --strip-components 1 


# Build a sysimage as llvm bitcode
WORKDIR /usr/share/julia/base
RUN julia --output-ji inference.ji coreimg.jl
RUN julia --output-ji sys.ji --sysimage inference.ji --startup-file=no sysimg.jl
ADD libjulia.bc /usr/share/julia/base/
ADD libuv.bc /usr/share/julia/base/
WORKDIR /work
ADD entrypoint.sh /work
ENTRYPOINT ["/work/entrypoint.sh"]

# docker run --rm -v $(pwd):/work -t jl2js example "['_myabs', '_myfun']"