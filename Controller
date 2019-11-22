package comPorts;

import javafx.beans.value.ChangeListener;
import javafx.fxml.FXML;
import javafx.scene.control.*;
import javafx.util.StringConverter;
import comPorts.enums.DataBits;
import comPorts.enums.Parity;
import comPorts.enums.StopBits;
import comPorts.enums.BoundRate;
import jssc.*;

import java.io.ByteArrayOutputStream;
import java.io.IOException;


public class Controller {
    @FXML
    private ComboBox<String> availablePorts;
    @FXML
    private ComboBox<BoundRate> availableBaudRate;
    @FXML
    private ComboBox<Parity> availableParity;
    @FXML
    private ComboBox<StopBits> availableStopBits;
    @FXML
    private ComboBox<DataBits> availableDataBits;
    @FXML
    private TextArea debugArea;
    @FXML
    private TextArea inputArea;
    @FXML
    private TextArea outputArea;
    @FXML
    private ToggleButton connectButton;
    @FXML
    private CheckBox errorSimulation;
    @FXML
    private TextField sourceField;
    @FXML
    private TextField destinationField;


    private final Integer FLAG = 14;

    private String packageMessage = "";

    private ChangeListener<String> listener = (observable, oldValue, newValue) -> {
        sendMessage(oldValue, newValue);
    };

    private PortReader portReader = new PortReader();

    private SerialPort serialPort;

    @FXML
    public void initialize() {
        initInputArea();
        availablePorts.getItems().addAll(SerialPortList.getPortNames());
        availablePorts.setConverter(new StringConverter<String>() {
            @Override
            public String toString(String object) {
                return object;
            }

            @Override
            public String fromString(String string) {
                return null;
            }
        });

        availableDataBits.getItems().addAll(DataBits.values());
        availableDataBits.setConverter(new StringConverter<DataBits>() {
            @Override
            public String toString(DataBits object) {
                return String.valueOf(object.getValue());
            }

            @Override
            public DataBits fromString(String string) {
                return null;
            }
        });

        availableBaudRate.getItems().addAll(BoundRate.values());
        availableBaudRate.setConverter(new StringConverter<BoundRate>() {
            @Override
            public String toString(BoundRate object) {
                return String.valueOf(object.getValue());
            }

            @Override
            public BoundRate fromString(String string) {
                return null;
            }
        });

        availableStopBits.getItems().addAll(StopBits.values());
        availableStopBits.setConverter(new StringConverter<StopBits>() {
            @Override
            public String toString(StopBits object) {
                return object.getName();
            }

            @Override
            public StopBits fromString(String string) {
                return null;
            }
        });

        availableParity.getItems().addAll(Parity.values());
        availableParity.setConverter(new StringConverter<Parity>() {
            @Override
            public String toString(Parity object) {
                return object.getName();
            }

            @Override
            public Parity fromString(String string) {
                return null;
            }
        });

        sourceField.clear();
        destinationField.clear();
        errorSimulation.setSelected(false);

        inputArea.setDisable(true);
    }

    private void initInputArea(){
        inputArea.textProperty().addListener((observable, oldValue, newValue) -> {
            if (!(newValue.length() < oldValue.length())) {
                if (inputArea.getCaretPosition() == inputArea.getText().length() - 1) {
                    char symbol = newValue.charAt(newValue.length() - 1);
                    if ((symbol >= 'а' && symbol <= 'я') || (symbol >= 'А' && symbol <= 'Я')) {
                        debugArea.appendText("Incorrect symbol");
                        return;
                    }
                    String message = String.valueOf(symbol);
                    packageMessage = packageMessage.concat(message);

                    if (packageMessage.length() == 7) {
                        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
                        try {
                            byteArrayOutputStream.write(FLAG);
                            byteArrayOutputStream.write(Integer.parseInt(destinationField.getText()));
                            byteArrayOutputStream.write(Integer.parseInt(sourceField.getText()));
                            byteArrayOutputStream.write(packageMessage.getBytes());
                            byteArrayOutputStream.write(errorSimulation.isSelected() ? 1 : 0);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        byte[] packet = BitStaffing.doBitStaffing(byteArrayOutputStream.toByteArray());
                        printDebug("Hex packet: 0x" + BitStaffing.bytesToHex(packet) );
                        try {
                            serialPort.writeBytes(packet);
                        } catch (SerialPortException e) {
                           printDebug("Error with sending message");
                        }
                        packageMessage = "";
                    }
                }
            }
        });
    }

    public boolean paramsIsCorrect() {
        if (availableParity.getValue() == null || availableStopBits.getValue() == null ||
                availableDataBits.getValue() == null || availableBaudRate.getValue() == null ||
                availablePorts.getValue() == null || sourceField.getText() == null ||
                destinationField.getText() == null) {
            printDebug("Choose all parameters");
            return false;
        }

        if (availableStopBits.getValue().equals(StopBits.STOP_BITS_2) &&
                availableDataBits.getValue().equals(DataBits.DATA_BITS_5)) {
            printDebug("If you choose 2 stop bits you can't use 5 data bits");
            return false;
        }

        if (availableStopBits.getValue().equals(StopBits.STOP_BITS_1_5) &&
                !availableDataBits.getValue().equals(DataBits.DATA_BITS_5)) {
            printDebug("If you choose 1.5 stop bits you can use only 5 data bits");
            return false;
        }
        try {
            Integer source = Integer.parseInt(sourceField.getText());
            Integer destination = Integer.parseInt(destinationField.getText());
            if(source < 0 || source > 255 ||
                    destination < 0 || destination > 255){
                printDebug("The size of addresses must be 1 byte");
                return false;
            }
            if(source.equals(destination)){
                printDebug("Source address and destination address must be not equals");
                return false;
            }
        }catch (NumberFormatException ex){
            printDebug("Addresses not a numbers");
            return false;
        }

        return true;
    }

    void printDebug(String message) {
        debugArea.appendText(message + "\n");
    }

    void disconnectFromPort() throws SerialPortException {
        serialPort.removeEventListener();
        serialPort.closePort();
        //serialPort = null;
    }

    @FXML
    public void connectToPortAction() {
        if (connectButton.getText().equals("Disconnect")) {
            try {
                disconnectFromPort();

                sourceField.setDisable(false);
                destinationField.setDisable(false);
                errorSimulation.setDisable(false);
                availablePorts.setDisable(false);
                availableBaudRate.setDisable(false);
                availableParity.setDisable(false);
                availableStopBits.setDisable(false);
                availableDataBits.setDisable(false);
                connectButton.setSelected(false);
                outputArea.clear();
                inputArea.textProperty().removeListener(listener);
                inputArea.clear();
                inputArea.setDisable(true);




                connectButton.setText("Connect");

                printDebug("Disconnected");

                return;
            } catch (SerialPortException e) {
                printDebug("Failed disconnection from port");
                return;
            }
        }
        if (!paramsIsCorrect()) {
            printDebug("Parameters wasn't set");
            connectButton.setSelected(false);
            return;
        }
        try {
            serialPort = new SerialPort(availablePorts.getValue());

            serialPort.openPort();

            serialPort.setParams(availableBaudRate.getValue().getValue(),
                    availableDataBits.getValue().getValue(),
                    availableStopBits.getValue().getValue(),
                    availableParity.getValue().getValue());
            printDebug("Parameters have been set");

            serialPort.setFlowControlMode(SerialPort.FLOWCONTROL_RTSCTS_IN |
                    SerialPort.FLOWCONTROL_RTSCTS_OUT);
            serialPort.addEventListener(portReader, SerialPort.MASK_RXCHAR);

            connectButton.setText("Disconnect");
            connectButton.setSelected(false);

            sourceField.setDisable(true);
            destinationField.setDisable(true);
            errorSimulation.setDisable(true);
            availablePorts.setDisable(true);
            availableBaudRate.setDisable(true);
            availableParity.setDisable(true);
            availableStopBits.setDisable(true);
            availableDataBits.setDisable(true);
            inputArea.setDisable(false);
            printDebug("Connected");
        } catch (SerialPortException ex) {
            printDebug("Failed to connect to port");
            connectButton.setSelected(false);
        }
    }

    private class PortReader implements SerialPortEventListener {
        @Override
        public void serialEvent(SerialPortEvent event) {
            if (event.isERR()) {
                printDebug("Error with parameters of COM ports");
                return;
            }
            if (event.isRXCHAR() && event.getEventValue() > 0) {
                try {
                    byte[] message;
                    message = serialPort.readBytes(event.getEventValue());
                    message = BitStaffing.doDeBitStaffing(message);
                    if(message[0] == FLAG && message[1] == Integer.parseInt(sourceField.getText()) &&
                            message[10] == 0) {
                        String str = new String(message);
                        outputArea.appendText(str.substring(3,10));
                    }else if(message[10] == 1 ) {
                        printDebug("Income packet has some errors");
                    }
                } catch (SerialPortException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    private void sendMessage(String oldValue, String newValue) {
        String message;
        if (newValue.length() > oldValue.length()) {
            if (inputArea.getCaretPosition() != inputArea.getText().length() - 1) return;
            message = newValue.substring(newValue.length() - 1);
        } else message = "\b";
        try {
            serialPort.setRTS(true);
            serialPort.writeBytes(message.getBytes());
            serialPort.setRTS(false);
        } catch (SerialPortException ex) {
            printDebug("Error sending message");
        }
    }

}
