#!/usr/bin/env pwsh

# The above line declared that this is a Powershell environment and should run with pwsh.
#  This lets us ./ it without it crying to us about having ./'d it.
  
# Import-Module here ensures that we have the library we need to talk to the GPIO pins
Import-Module -Name Microsoft.PowerShell.IoT

# The pin numbers are using WiringPi numbers, so they do not make sense
#  at all which one is which on the board. I've written which one they
#  physically are on the board though so you know where to put the wires.
$relay1Pin = 0 # This one is on physical pin 11
$relay2Pin = 2 # This one is on physical pin 13

# The function below is to make the usage sort of "generic use". The function takes a
#  pin and the action we want to provide ability with, in this case, push. Like pushing
#  a button attached to the pin.
function Push-Button ($pin, $action) {
  # If the action is push, we will set the pin to Low if it is High and vice versa, wait
  #  one second, then set it back. Otherwise it'd be like holding it down for however
  #  many seconds we put on here, or forever, without a timer, until it lost power.
  #  If we don't get the corresponding options given to us when this is called, we will
  #  error out onto the screen with the Write-Host cmdlet stating what we did not
  #  understand.
  if ($action -eq "push") {
    if ((Get-GpioPin -Id $pin).Value -eq "High") {
      Set-GpioPin -Id $pin -Value "Low"
      Start-Sleep -Seconds 1
      Set-GpioPin -Id $pin -Value "High"
    } elseif ((Get-GpioPin -Id $pin).Value -eq "Low") {
      Set-GpioPin -Id $pin -Value "High"
      Start-Sleep -Seconds 1
      Set-GpioPin -Id $pin -Value "Low"
    } else {
      Write-Host "Could not determine pin $pin`'s value status. function Push-Button."
    }
  } else {
    Write-Host "Could not determine action provided. function Push-Button."
  }
}

# Since I like to record data, and I have a bunch of free space on the SD card, we will
#  log the time that the "button" was pushed and write it out to a log.
$date = Get-Date -Format "yyyy`/MM`/dd hh`:mm`:ss tt"
"$date - Garage Door button was pushed." | Out-File "~/garageDoor.log" -Append
# Now we will actually call the function.
Push-Button $relay2Pin "push"
