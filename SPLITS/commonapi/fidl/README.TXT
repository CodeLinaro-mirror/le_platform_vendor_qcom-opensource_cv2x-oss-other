1. Generate commonapi core source
    1.1 commonapi-core-generator-linux-x86_64 SplitsConfig.fidl -sk -d core
    1.2 commonapi-core-generator-linux-x86_64 VehicleData.fidl -sk -d core
    1.3 commonapi-core-generator-linux-x86_64 common-core.fdepl -sk -d core

2 Generate vsomeip binding code
    commonapi-someip-generator-linux-x86_64 -d someip Splits-SomeIp.fdepl

