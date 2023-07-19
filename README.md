# SwiftUI-UITextView-text-and-image
SwiftUI Text Image
# The first part
![IMG_0321](https://github.com/S-way520/SwiftUI-UITextView-text-and-image/assets/95877651/71e2d378-cedd-4c5f-b690-696d8a217996)
# The second part
Code file 1
```swift
import SwiftUI
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```
Code file 2
```swift
import SwiftUI
struct ContentView: View {
    @State private var height: CGFloat = 100
    var body: some View {
        ScrollView {
            TextView(height: $height)
                .frame(height: height, alignment: .center)
                .border(Color.red, width: 1)
                .padding(10)
        }
    }
}
protocol TextEditorDelegate: AnyObject { func insertImage() }
struct TextView: UIViewControllerRepresentable {
    //
    @Binding private var height: CGFloat
    //
    private var controller: UIViewController
    private var textView: UITextView
    private var accessoryView: InputImageView
    init(height: Binding<CGFloat>) {
        self._height = height
        self.controller = UIViewController()
        self.textView = UITextView()
        self.accessoryView = InputImageView()
    }
    func makeUIViewController(context: Context) -> some UIViewController {
        textView.text = "默认"
        textView.font = UIFont.systemFont(ofSize: 48)
        textView.textColor = UIColor.green
        textView.textAlignment = .left
        textView.selectedRange = NSMakeRange(0, 2)
        textView.isEditable = true
        textView.isSelectable = true
        textView.allowsEditingTextAttributes = true
        textView.isScrollEnabled = false
        textView.isUserInteractionEnabled = true
        textView.backgroundColor = .clear
        textView.textContainer.lineFragmentPadding = 0
        textView.textContainerInset = UIEdgeInsets.zero//(top: 50, left: 50, bottom: 50, right: 50)
        textView.layoutManager.allowsNonContiguousLayout = false
        
        textView.setContentCompressionResistancePriority(.defaultLow, for: .horizontal)
        controller.view.addSubview(textView)
        textView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            textView.centerXAnchor.constraint(equalTo: controller.view.centerXAnchor),
            textView.centerYAnchor.constraint(equalTo: controller.view.centerYAnchor),
            textView.widthAnchor.constraint(equalTo: controller.view.widthAnchor),
        ])
        textView.delegate = context.coordinator
        context.coordinator.textViewDidChange(textView)//Height text
        accessoryView.delegate = context.coordinator
        textView.inputAccessoryView = accessoryView
        return controller
    }
    func updateUIViewController(_ uiViewController: UIViewControllerType, context: Context) { }
    func makeCoordinator() -> Coordinator { Coordinator(self) }
    
    class Coordinator: NSObject, UITextViewDelegate, UIImagePickerControllerDelegate, UINavigationControllerDelegate, TextEditorDelegate {
        var parent: TextView
        init(_ parent: TextView) {
            self.parent = parent
        }
        func insertImage() {
            let imagePicker = UIImagePickerController()
            imagePicker.delegate = self
            imagePicker.allowsEditing = true
            parent.controller.present(imagePicker, animated: true, completion: nil)
        }
        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            if let pickerImage = info[UIImagePickerController.InfoKey.editedImage] as? UIImage {
                let newString = NSMutableAttributedString(attributedString: parent.textView.attributedText)
                newString.append(NSAttributedString(attachment: NSTextAttachment(image: pickerImage)))
                parent.textView.attributedText = newString
                textViewDidChange(parent.textView)//Height image
            }
            picker.dismiss(animated: true, completion: nil)
        }
        //
        func textViewDidChange(_ textView: UITextView) {
            let size = CGSize(width: parent.controller.view.frame.width, height: .infinity)
            let estimatedSize = textView.sizeThatFits(size)
            if parent.height != estimatedSize.height {
                DispatchQueue.main.async {
                    self.parent.height = estimatedSize.height
                }
            }
            textView.scrollRangeToVisible(textView.selectedRange)
            textView.font = UIFont.systemFont(ofSize: 48)
            textView.textColor = UIColor.green
        }
        //
    }
}

class InputImageView: UIInputView {
    weak var delegate: TextEditorDelegate!
    init() {
        super.init(frame: CGRect(x: 0, y: 0, width: 50, height: 25), inputViewStyle: .default)
        setupImage()
    }
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    func setupImage() {
        let insert = UIButton()
        insert.setImage(UIImage(systemName: "photo.on.rectangle.angled"), for: .normal)
        insert.addTarget(self, action: #selector(insertImage(_:)), for: .touchUpInside)
        addSubview(insert)
        insert.translatesAutoresizingMaskIntoConstraints = false
    }
    @objc private func insertImage(_ button: UIButton) {
        delegate.insertImage()
    }
}
```
