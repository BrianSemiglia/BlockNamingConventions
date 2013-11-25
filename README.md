Objective-C Block Argument Format Convention
======================

Just like delegation, blocks are used as a means of callback. A preferred convention for delegate interfaces is to include the messaging object in the message as an argument.

    - (void)objectDidReceiveEvent:(id)object;

This allows the delegate to distinguish the messaging object and respond with any necessary changes specific to it. With blocks this style is also necessary but for different reasons. Take for example the following method:

    - (void)performMethodWithCompletionHandler:(void (^)(NSArray *results, NSError *error)completionHandler;
    
In practice, there would be less confusion about the object that is performing the callback because of the inline style. However, if the completionHandler was a strong property, referring to the object within the block would cause a retain cycle:

    [object performMethodWithCompletionHandler:^(NSArray *results, NSError *error) {        
         // Referencing object inside of object's completionHander creates retain cycle.
         [object reset];
    }];
    
This can be solved with the following:

    __weak Object *weakObject = self.object;
    [self.object performMethodWithCompletionHandler:^(NSArray *results, NSError *error) {
        // Using a weak pointer prevents the block from retaining object and prevents the retain cycle.
        [weakObject reset];
    }];
    
However, there is a better solution:

     [self.object performMethodWithCompletionHandler:^(Object *object, NSArray *results, NSError *error) {
         // The object pointer within this scope refers to the argument passed in by the block and thus prevents the retain cycle.         
         [object reset];
     }];
     
This solution is an especially nice addition for those targeting 4.3 and do not have the option of using __weak.
