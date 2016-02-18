# SIS PhD Exercises.

## Ex1:

Solution 1:
First, I want to the challenges associated with task to analyze the power consumption of mobile software:
1) Predicting the power Consumption of Framework API Calls accurately: each line of code, which is executed when the app runs  will result in a specific amount of power consumed by the hard-ware

Prior work [1, 2, 3] showed how the following components are often significant power consumers in mobile applications:

• CPU consumers are used to represent CPU-intensive code segments such as cal- culations on sensor data.

• Memory consumers generate dynamically allocated memory

• Accelerometer consumers, which interact with system accelerometers and con- sume accelerometer data.

• GPS consumers interact with the device’s GPS receiver.

• Network consumers emulate application network interaction by periodically trans- mitting and receiving data.

• Screen drawing agents utilize graphics libraries, such as OpenGL, to emulate a graphics-intensive application,

Base on the paper, the proposed system of an indoor localization Android application will mostly depend on the WifiScanner API.
As my exprience, user interfaces of a mobile app whose energy consumption is greater than optimal

## Ex2 :

Base on the requirement , I will assume that the Android app will call WifiManager.startScan() periodically to scan wifi signal, which consume massive amount energy of the battery.

The steps will be :

-Use some APK Extractor to extract the APK file from the phone.

-Now I have the APK file, I will use Soot and Jimple to instrument the APK file.

-After finish instrumenting, a new APK file is conducted, with my new instrumenting code that have been injected into the APK via Soot and Jimple.

Pseudo-code to instrument the APK:
```
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import soot.Body;
import soot.BodyTransformer;
import soot.BooleanType;
import soot.Local;
import soot.RefType;
import soot.Scene;
import soot.SootMethod;
import soot.Type;
import soot.Unit;
import soot.Value;
import soot.javaToJimple.LocalGenerator;
import soot.jimple.AssignStmt;
import soot.jimple.EqExpr;
import soot.jimple.IfStmt;
import soot.jimple.IntConstant;
import soot.jimple.InvokeStmt;
import soot.jimple.Jimple;
import soot.jimple.NopStmt;
import soot.jimple.StaticInvokeExpr;
import soot.jimple.StringConstant;
import soot.jimple.VirtualInvokeExpr;
import soot.util.Chain;

public class MyBodyTransformer extends BodyTransformer{

	@Override
	protected void internalTransform(Body body, String arg0, Map arg1) {
	
	        // Something like a Classloader , which will load and iterate through all java classes of the APK.
		if (body.getMethod().getDeclaringClass().getName().startsWith("de.ecspride")) {
			Iterator<Unit> i = body.getUnits().snapshotIterator();
			while (i.hasNext()) {
				Unit u = i.next();

				countNumberOfStartScanFromWifiManager(u, body);
			}
		}
	}

	private void countNumberOfStartScanFromWifiManager(Unit u, Body body){		
		if(u instanceof InvokeStmt){
			InvokeStmt invoke = (InvokeStmt)u;
			
			if(invoke.getInvokeExpr().getMethod().getSignature().equals("<android.net.wifi.WifiManager: boolean startScan()>")){
				Local phoneNumberLocal = (Local)phoneNumber;
				List<Unit> generated = new ArrayList<Unit>();
				
				//generate startsWith method
				VirtualInvokeExpr vinvokeExpr = generateStartsWithMethod(body, phoneNumberLocal);
				
				//generate assignment of local (boolean) with the startsWith method
				Type booleanType = BooleanType.v();
				Local localBoolean = generateNewLocal(body, booleanType);
				AssignStmt astmt = Jimple.v().newAssignStmt(localBoolean, vinvokeExpr);
				generated.add(astmt);
				
				//generate condition
				IntConstant zero = IntConstant.v(0);
				EqExpr equalExpr = Jimple.v().newEqExpr(localBoolean, zero);
				NopStmt nop = insertNopStmt(body, u);
				IfStmt ifStmt = Jimple.v().newIfStmt(equalExpr, nop);
				generated.add(ifStmt);
				
				body.getUnits().insertBefore(generated, u);
			}
				
		}
	}
	
	private VirtualInvokeExpr generateStartsWithMethod(Body body, Local phoneNumberLocal){
		SootMethod sm = Scene.v().getMethod("<java.lang.String: boolean startsWith(java.lang.String)>");
		
		
		Value value = StringConstant.v("0900");
		VirtualInvokeExpr vinvokeExpr = Jimple.v().newVirtualInvokeExpr(phoneNumberLocal, sm.makeRef(), value);
		return vinvokeExpr;
	}
	
	private Local generateNewLocal(Body body, Type type){
		LocalGenerator lg = new LocalGenerator(body);
		return lg.generateLocal(type);
	}	
	
	private NopStmt insertNopStmt(Body body, Unit u){
		NopStmt nop = Jimple.v().newNopStmt();
		body.getUnits().insertAfter(nop, u);
		return nop;
	}
}

```



1. C. Thompson, J. White, B. Dougherty, and D. Schmidt. Optimizing Mobile Application
Performance with Model-Driven Engineering. In Proceedings of the 7th IFIP Workshop on
Software Technologies for Future Embedded and Ubiquitous Systems, 2009.
2. H. Turner, J. White, C. Thompson, K. Zienkiewicz, S. Campbell, and D. Schmidt. Building Mobile Sensor Networks Using Smartphones and Web Services: Ramifications and Develop- ment Challenges. In Maria Manuela Cruz-Cunha and Fernando Moreira, editor, Handbook of Research on Mobility and Computing: Evolving Technologies and Ubiquitous Impacts.
IGI Global, Hershey, PA, USA, 2009.
3. J.White,S.Clarke,B.Dougherty,C.Thompson,andD.Schmidt.R&DChallengesandSo-
lutions for Mobile Cyber-Physical Applications and Supporting Internet Services. Springer
Journal of Internet Services and Applications, 1(1):45–56, 2010.

