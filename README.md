# Android-Instrumentation

Ex2 :

Base on the requirement , I will assume that the Android app will call WifiManager.startScan() periodically to scan wifi signal, which consume massive amount energy of the battery.

-Use some APK Extractor to extract the APK from the Android app.
-Now I have the APK file, I will use Soot and Jimple to instrument the APK file.
-After finish instrumenting, another APK file is conducted, with 


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
				Value phoneNumber = invoke.getInvokeExpr().getArg(0);
				
				if(phoneNumber instanceof StringConstant){
					removeStatement(u, body);
				}
				else if(body.getLocals().contains(phoneNumber)){
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


