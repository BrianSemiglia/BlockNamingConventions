BlockNamingConventions
======================

Objective-C Block Naming Conventions

Just like delegation, blocks are used as a means of callback. A preferred convention for delegate interfaces is to include the messaging object in the message.

    - (void)objectDidReceiveEvent:(id)object;

This allows the delegate to distinguish the messaging object and apply any necessary changes to it.

With blocks this style is also necessary but for different reasons. Take for example the following method:

    - (void)performMethodWithCompletionHandler:(void (^)(NSArray *results, NSError *error)completionHandler;
    
In practice, there would be no confusion about the object that is performing the callback. However if the completionHandler was a strong property of object, referring to the object would cause a retain cycle:

    [object performMethodWithCompletionHandler:^(NSArray *results, NSError *error) {        
         NSLog("%@", results);
     
         // Referencing object inside of object's completionHander creates retain cycle.
         [object reset];
    }];
    
This can be solved with the following:

    __weak Object *weakObject = object;
    [object performMethodWithCompletionHandler:^(NSArray *results, NSError *error) {
        NSLog("%@", results);
     
        // Using a weak pointer prevents the block from retaining object and prevents the retain cycle.
        [weakObject reset];
    }];
    
However, there is a better solution:

     [object performMethodWithCompletionHandler:^(Object *object, NSArray *results, NSError *error) {
         NSLog("%@", results);

         // The object pointer within this scope refers to the argument passed in by the block and thus prevents the retain cycle.         
         [object reset];
     }];
     
This solution is an especially nice addition for those targeting 4.3 and do not have the option of using __weak.
