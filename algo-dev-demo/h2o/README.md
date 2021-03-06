#### Before you begin
1. Create an account on [algorithmia.com](https://algorithmia.com/) and take note of your username (referred to in the rest of these instructions as YOUR_USERNAME)

#### Download the demo .jar and .java files
1. Download [h2o-genmodel.jar](https://github.com/algorithmiaio/sample-apps/raw/master/algo-dev-demo/h2o//h2o-genmodel.jar), [gbm_pojo_test.java](https://github.com/algorithmiaio/sample-apps/raw/master/algo-dev-demo/h2o/gbm_pojo_test.java), and [main.java](https://github.com/algorithmiaio/sample-apps/raw/master/algo-dev-demo/h2o/main.java) from this repository
2. Alternately, you can generate them yourself by following steps 1-3 of H2O's [POJO Quick Start](https://h2o-release.s3.amazonaws.com/h2o/rel-turing/1/docs-website/h2o-docs/pojo-quick-start.html)

#### Create your algorithm and pull down via Git
1. Click the "+" on [algorithmia.com](https://algorithmia.com/) to create a new algorithm
2. Give your algorithm any name (referred to in the rest of these instructions as YOUR_ALGO_NAME), pick "Java", "Full access to internet" and "Can call other algorithms" (not "Advanced GPU")
3. Click Create Algorithm
4. Copy the Git command, open up a local console in a directory of your choosing, and run the Git command to pull down the algorithm's repo

Note: you can always find the Git command in the "manage" tab of [your algorithm's webpage](https://algorithmia.com/user#))

#### Modify/add files to the algorithm repo
1. Copy `gbm_pojo_test.java` into `src/main/java/YOUR_USERNAME/YOUR_ALGO_NAME/` and add a package declaration so that the first line reads `package YOUR_USERNAME.YOUR_ALGO_NAME;`
2. create a folder `lib` at the root of the repo and copy `h2o-genmodel.jar` into it
3. modify `pom.xml` to add that jar as a local dependency, by pasting this xml snippet into the end of the `dependencies` element:
```
        <!-- local library dependency: insert before </dependencies> in your pom.xml file -->
        <dependency>
            <groupId>hex.genmodel</groupId>
            <artifactId>genmodel</artifactId>
            <version>1.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/lib/h2o-genmodel.jar</systemPath>
        </dependency>
```

#### Copy code from main.java into YOUR_ALGO_NAME.java
1. Open `YOUR_ALGO_NAME.java`, which was autogenerated when you created your algo; and copy the following code from `main.java` into it:
- copy over all import statements (`java.io.*`, `hex.genmodel.easy.RowData`, `hex.genmodel.easy.EasyPredictModelWrapper`, `hex.genmodel.easy.prediction.*`)
- copy the `modelClassName` into your `public class YOUR_ALGO_NAME {...`, but alter it to read `private static String modelClassName = "YOUR_USERNAME.YOUR_ALGO_NAME.gbm_pojo_test";`
- alter the `apply` method to return a Map
- copy the entire body (but not method declaration) of `main(String[] args)` into your `apply()` method
- remove all `System.out.print` and `println` calls and place their output into a Map, and return this Map; for example:
```
    BinomialModelPrediction p = model.predictBinomial(row);
    HashMap outs = new HashMap();
    outs.put("prediction",p.label);
    outs.put("probabilities", p.classProbabilities);
    return outs;
```
Note: once you've done this, you can delete `main.java`, since the `apply()` method of `YOUR_ALGO_NAME.java` is now the algorithm's main entry point

#### Publish your algorithm
1. Git Add, Commit, and Push your changes
2. Head back to your algorithm's webpage and click the "Source" tab to start up the Web IDE
3. Click "Compile" and wait for success (ignore any warnings about "dependency.SystemPath"), then test your algo by giving it empty String input (`""`) on the Web Console below the IDE
4. Click "Publish" and walk through the dialogs
5. Test out your new Algorithm by clicking "Run Example"; you should get back a result like `{"Prediction":"YES","Probability 0":0.4319916897116479,"Probability 1":0.5680083102883521}`
6. Write some brief documentation in the "Docs" tab of your algorithm's webpage (or the `README.md` file in its repo)

#### Alter your algorithm to accept inputs
1. Alter the `apply` method to accept a Map (`apply` does not support overloading, so instead we'll use a Map to pass parameters)
2. Remove the hard-coded values for "Year", "Month", etc and extract them from the Map instead (e.g. `input.get("Year")`)
3. Compile, then test: in the Web IDE, pass a JSON Object like `{"Year":"1987", "Month":"10", "DayOfMonth":"14", "DayOfWeek":"3", "CRSDepTime":"730", "UniqueCarrier":"PS", "Origin":"SAN", "Dest":"SFO"}`
4. Publish and document these changes

Note: if you get a `NullPointerException` at this last step, it likely indicates that a required parameter (e.g. `s.get("Year")`) was missing from the input; to help the user avoid this problem, check your parameters before using them:
```
String[] params = {"Year","Month","DayOfMonth","DayOfWeek","CRSDepTime","UniqueCarrier","Origin","Dest"};
for (String k: params) {
  if(s.get(k)==null) {
    throw new Exception(k+" is missing");
  }
}
```

You can view a completed (and slightly more optimized) version of this algorithm at [algorithmia.com/algorithms/jpeck/h2o_demo](https://algorithmia.com/algorithms/jpeck/h2o_demo) and inspect its source code via the "Source" tab