@app.route("/play/<pin>")
def play(pin): return render_template_string(STUDENT_HTML,pin=pin,invalid=pin not in games)
@app.route("/logout")
def logout(): session.clear(); return redirect("/")
@socketio.on("teacher_join")
def tj(d):
 pin=d.get("pin"); g=games.get(pin)
 if not g or g["teacher"]!=session.get("teacher"): return
 join_room(pin); emit("lobby",{"players":[p for p in g["players"].values() if not p["removed"]]})
@socketio.on("student_join")
def sj(d):
 pin=d.get("pin"); g=games.get(pin); nick=str(d.get("nick","")).strip()[:20]
 if not g or g["status"]!="lobby": emit("join_error","PIN noto‘g‘ri yoki o‘yin boshlangan."); return
 if not nick: emit("join_error","Nik kiriting."); return
 g["players"][request.sid]={"nick":nick,"avatar":d.get("avatar","🤖"),"score":0,"answers":[],"exits":0,"removed":False}
 join_room(pin); emit("joined",{"nick":nick}); socketio.emit("lobby",{"players":[p for p in g["players"].values() if not p["removed"]]},to=pin)
@socketio.on("start")
def start(d):
 pin=d.get("pin"); g=games.get(pin)
 if not g or g["teacher"]!=session.get("teacher"): return
 g["status"]="playing"; g["index"]=0; sendq(pin)
def sendq(pin):
 g=games[pin]; q=g["questions"][g["index"]]
 socketio.emit("question",{"n":g["index"]+1,"total":10,"q":q[0],"options":q[1]},to=pin)
@socketio.on("answer")
def answer(d):
 pin=d.get("pin"); g=games.get(pin); p=g and g["players"].get(request.sid)
 if not p or p["removed"] or g["status"]!="playing" or g["index"] in p["answers"]: return
 ok=int(d.get("choice",-1))==g["questions"][g["index"]][2]; p["answers"].append(g["index"])
 if ok: p["score"]+=1000
 emit("feedback",{"ok":ok,"score":p["score"]})
@socketio.on("next")
def nextq(d):
 pin=d.get("pin"); g=games.get(pin)
 if not g or g["teacher"]!=session.get("teacher"): return
 g["index"]+=1
 if g["index"]>=10:
  g["status"]="finished"; ranking=sorted([p for p in g["players"].values() if not p["removed"]],key=lambda p:p["score"],reverse=True)
  socketio.emit("finished",{"ranking":ranking},to=pin)
 else: sendq(pin)
@socketio.on("tab_exit")
def tab_exit(d):
 g=games.get(d.get("pin")); p=g and g["players"].get(request.sid)
 if not p or p["removed"]: return
 p["exits"]+=1
 if p["exits"]>=2: p["removed"]=True; emit("removed","Ikkinchi marta chiqdingiz. O‘yindan chiqarildingiz.")
 else: emit("warning","Ogohlantirish: sahifadan chiqib ketmang. Yana chiqilsa o‘yindan chiqarilasiz.")
 socketio.emit("lobby",{"players":[x for x in g["players"].values() if not x["removed"]]},to=d.get("pin"))
if __name__=="__main__": socketio.run(app,host="0.0.0.0",port=int(os.environ.get("PORT",5000)),debug=False)
