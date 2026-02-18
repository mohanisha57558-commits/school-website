from flask import Flask, render_template, request, redirect, url_for, session, send_from_directory
import os

app = Flask(__name__)
app.secret_key = "secret123"

UPLOAD_FOLDER = "uploads"
app.config["UPLOAD_FOLDER"] = UPLOAD_FOLDER

ADMIN_USERNAME = "admin"
ADMIN_PASSWORD = "1234"

@app.route("/")
def home():
    return render_template("view.html")

@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        if request.form["username"] == ADMIN_USERNAME and request.form["password"] == ADMIN_PASSWORD:
            session["admin"] = True
            return redirect(url_for("admin"))
    return render_template("login.html")

@app.route("/admin", methods=["GET", "POST"])
def admin():
    if not session.get("admin"):
        return redirect(url_for("login"))

    if request.method == "POST":
        file = request.files["file"]
        if file:
            file.save(os.path.join(app.config["UPLOAD_FOLDER"], "timetable.pdf"))

    return render_template("admin.html")

@app.route("/uploads/<filename>")
def uploaded_file(filename):
    return send_from_directory(app.config["UPLOAD_FOLDER"], filename)

@app.route("/logout")
def logout():
    session.pop("admin", None)
    return redirect(url_for("home"))

if __name__ == "__main__":
    app.run()
